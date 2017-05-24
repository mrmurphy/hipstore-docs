# Adding the Cart Page

Stuff's about to get real. Almost any real-world single-page app will \(contradictorally\) have more than one page. Which page is showing at any given moment is determined by the path in the location bar. In our case we'll be abusing the hash portion of the path, which is normally used to tell the browser where to scroll to, by instead using it to determine which page to show. We're only using the hash right now out of a lazy desire not to modify the server routing at all. In a real production app we'd most likely use the whole path for routing instead of just the hash, and we'd modify the server so that the paths for each of our pages all direct to the same html and javascript files that run our app.

Here's what our routes will look like:

```
/# (or /)   <- The products page
/#cart       <- The cart page
```

We'll just be doing the basics today. Routes can have all kinds of fancy nesting and parameters and things, but we're just going to stick with basic page switching right now.

## Setting Up the Router

### What we need to make

In order to set up routing, we'll need the following things:

* A type union \(we'll call it `Page`\) with a constructor for each page
* Parsers for each route
* A function that turns a `Navigation.Location` into a `Page`
* A function that turns a `Page` into a `Cmd msg` and navigates to a new page

### What we need to do

Once we've got those things, we'll need to:

* Use `Navigation.program` instead of `Html.program`. This will pass the location into your init function, and it'll call your update function whenever the location changes.
* Use the functions above to store the page on the model and determine which page should be displayed
* Add a new message for changing pages, and use the function above to get it done.

### Getting it done

#### The Page type

First, let's mark off a new section of the file with a `--- Router ---` comment, and add in a new type for our pages:

```
--- Router ---


type Page
    = Home
    | Cart
```

Simple enough!

#### The route parsers

Next, we'll build a parser for each route, and for the sake of organization, we'll stick them into a record together:

\(You'll need to add `import Route exposing ((:=))` in order for this to work\)

```
-- I've left off the type signature on purpose because it's ugly and big 💄
routeParsers =
    { home = Home := Route.static ""
    , cart = Cart := Route.static "#cart"
    }
```

Let's take a moment here and examine what's what. Also, we're using a nice library here to help us with route parsing called `Bogdanp/elm-route`. It exposes a module called `Route` with some useful functions inside.

Looking at the Cart route parser:

```
Cart := Route.static "#cart"
  ☝️  ☝️     ☝️          ☝️
 (A) (B)    (C)        (D)
```

* \(A\) Is the type constructor for the route page. We're just defining that when this parser succeeds, we want to get the Cart page out of it.
* \(B\) `:=` is an infix function from `Route` that takes a type constructor on one side, and parsing rules on the right side.
* \(C\) `Route.static` is a function from `Route` that tells us that we're about to pass in a part of the path that's a fixed string, it's not a dynamic parameter.
* \(D\) is the very string that we referenced in C. It's the hash path that we want to represent our route.

These parsers will be used below to turn a string into a Page, and vice-versa.

#### Location to Page function

`Route` provides a type `Router` that will let us take a string and turn it into a `Page`.  Here we build up a `Router` by using our two parsers from above:

```
router : Route.Router Page
router =
    Route.router
        [ routeParsers.home
        , routeParsers.cart
        ]
```

And then we use that router to build a function that, when given a location, will get the path and hash, parse it, and return a page.

```
pageFromLocation : Location -> Page
pageFromLocation location =
    (location.pathname ++ location.hash)
        |> Route.match router
        |> Maybe.withDefault Home
```

We'll use this function in a bit to store the current page on the model.

#### Page to Cmd function

The `Route` library provides a cool function called `reverse` that, when given a `Page` and any parameters the page requires, will produce a string. We can then use that string and the `Navigation` library to trigger a location change:

```
navigateTo : Page -> Cmd msg
navigateTo page =
    (case page of
        Home ->
            Route.reverse routeParsers.home []

        Cart ->
            Route.reverse routeParsers.cart []
    )
        |> Navigation.newUrl
```

#### Navigation.program

Now that we've got all the parts, let's put it together.

First stop is the `main` function. Let's change `Html.program` to `Navigation.program` \(we'll have to import `Navigation`, as well\).

```elm
main : Program Never Model Msg
main =
    Navigation.program
        { view = view
        ...
```

But wait! This will give us some compiler errors because `Navigation.program` takes an extra parameter, a function that takes a `Location` and turns it into a message. This message will be given to the update function whenever the location changes, so let's go add that message now:

```elm
...

type Msg
    = NoOp
    ...
    | LocationChanged Location


update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        NoOp ->
            model ! []

        ...

        LocationChanged location ->
            { model | page = pageFromLocation location } ! []

...

main =
    Navigation.program LocationChanged
        { view = view
        ...
```

You'll notice that we used `pageFromLocation` to turn the location into a page. Yay! But now we're trying to store the page on the model, and there's no field for that yet, so let's add that to the model type:

```
type alias Model =
    { products : WebData (List Product)
    ...
    , page : Page
    }
```

And we'll need to fix the init function, too. But what should the initial value of `page` be?

Well, it turns out that `Navigation.program` actually passes a `Location` to the init function as well, so we can just use that to get our first page:

```
init : Location -> ( Model, Cmd Msg )
init location =
    ( { products = RemoteData.NotAsked
        ...
      , page = pageFromLocation location
      }
...
```

Great job! Now if you compile, you shouldn't get any errors. But nothing has changed in the UI yet. We're still just showing the product list no matter what the path is.

#### Displaying the right page

Let's update the view function to render the correct page based on the model:

```
view : Model -> Html Msg
view model =
    div []
        [ case model.page of
            Home ->
                HipstoreUI.products <| uiConfig model

            Cart ->
                HipstoreUI.cart <| uiConfig model
        ]
```

We've added a case statement inside the wrapper div that renders the cart if that's the active page, or the product list if the home page is active. Let's test it by changing the URL by hand to `/#cart`:

![](/assets/import8.png)

#### Changing pages

Last thing we need is to make those navigation buttons work. Let's add a new message:

```
type Msg
    = NoOp
      ...
    | ChangePage Page

...

update msg model =
        ...
        ChangePage page ->
            model ! [ navigateTo page ]
```

Now we've got a message that we can use to change to a new page. Note that we didn't update the `page` field on the model. We want the change to go all the way out to the location bar, not just to the in-memory model, so we'll use the `navigateTo` function that we set up earlier.

Now we can just hook it up in the HUI config:

```
uiConfig : Model -> HipstoreUI.Config Msg
uiConfig model =
    { onAddToCart = AddToCart
      ...
    , onClickViewCart = ChangePage Cart
    , onClickViewProducts = ChangePage Home
      ...
    }
```

And you should now be able to click between pages. Congratulations!!!

🎉

