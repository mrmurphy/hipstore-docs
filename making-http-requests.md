# Adding the Cart Page

Stuff's about to get real. Almost any real-world single-page app will \(ironically\) have more than one page. Which page is showing at any given moment is determined by the path in the location bar. In our case we'll actually be abusing the hash portion of the path, which is normally used to tell the browser where to scroll to, and instead determine which page to show. We're only using the hash right now out of a lazy desire not to modify the server at all. In a real production app we'd most likely use the whole path for routing instead of just the hash, and we'd modify the server so that the paths for each of our pages all direct to the same html and javascript files that run our app.

Here's what our routes will look like:

```
/#/ (or /)   <- The products page
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
* \(C\) `Route.static` is a function from `Route` that tells us that we're about to pass in a part of the path that's a fixed string, it's not parameterized.
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

#### Displaying the right page

#### Changing pages



