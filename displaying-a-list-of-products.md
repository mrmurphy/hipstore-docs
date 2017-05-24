# Displaying a List of Products

---

## Fetching the Products

In Elm, an HTTP request is considered a _side effect_, or something that interacts with the outside world. Elm deals with side effects by wrapping them up as a piece of data _that can be run in the future_, and handing that data to the runtime \(Elm's internals\). When the data is handed off to the runtime, the programmer also tells the runtime what messages to send to the update function if/when the effect finishes. This piece of data that represents an effect is called a _Command_ or `Cmd` for short.

The basic steps we'll be following here are:

* Define what the HTTP request we want to send looks like
* Turn it into a command
* Make a place on the model for the data that we're expecting the request to return
* Make a message that the runtime can use when the HTTP request finishes
* Hand the command off to the runtime

### Remote Data

There's a package for representing data that resides somewhere outside of the app's memory called `krisajenkins/remotedata` this package makes it easy to keep track of the fact that the data could be in one of many different states at any given time. It could be loading, the request to load it could have errored out, we may not have even asked for the data yet, or it could be that everything is fine and we have the data immediately available.

You don't have to worry much about this library or data type in this workshop. It's enough to know its purpose, and that `hipstore-ui` requires it. As long as we make our types match up with what's required, we'll be fine.

### Constructing an HTTP Request

We can use the `elm-lang/http` package to build a representation of an http request that'll be sent later. First, at the top of the imports section, import the package:

```elm
import Http
```

Next, let's add a new section called "REQUESTS" above the "MODEL" section that already exists in the file, and declare a new value called `getProducts`:

```elm
---- REQUESTS ----

getProducts : Cmd Msg
getProducts =
    Http.get "https://hipstore.now.sh/api/products" (Json.Decode.list decodeProduct)

---- MODEL ----
...
```

This is a good start, but you'll get some compiler errors. Here's a quick list of ways to get through them:

* Import `Json.Decode`
* define `decodeProduct` somewhere in the file like this:

  ```
  decodeProduct: Json.Decode.Decoder Product
  decodeProduct = Debug.crash "TODO".
  ```

  Don't worry, we'll come back to that in a later section.

* Fix the definition of `getProducts` to match its definition \(see immediately below\)

### Turning the request into a command

Because we've specified that the type of `getProducts` is a `Cmd Msg`, which is what we want it to be in order to send it to the runtime \(side effects, remember?\) but the current type of the value is an \`Http.Request ...\`, we're getting a compiler error.

Conveniently, RemoteData comes with a function to do just that. Let's add it on with a pipe:

```elm
---- REQUESTS ----

getProducts : Cmd Msg
getProducts =
    Http.get "https://hipstore.now.sh/api/products" (Json.Decode.list decodeProduct)
        |> RemoteData.sendRequest

---- MODEL ----
...
```

Almost there. Now we've got a `Cmd (WebData (List Product))`. But, in order for the runtime to call our `update` function when the request completes, we've got to give it a command that contains a message \(`Cmd Msg`\), not a command containing data. But we don't have a message like that yet, so let's create one:

```elm
...
---- UPDATE ----


type Msg
    = NoOp
    | ProductsChanged (WebData (List Product)) -- <-- added this!


update : Msg -> Model -> ( Model, Cmd Msg )
...
```

and now we can add another pipe that'll map that command, wrapping its data up in our new message:

```elm
---- REQUESTS ----

getProducts : Cmd Msg
getProducts =
    Http.get "https://hipstore.now.sh/api/products" (Json.Decode.list decodeProduct)
        |> RemoteData.sendRequest
        |> Cmd.map ProductsChanged

---- MODEL ----
...
```

We save the file with high hopes, only to find out that the compiler has something else to do. Now we've got to fix the `update` function.

### The Model & Update

So we're intending to fetch a list of products from the server, and then give them to `hipstore-ui` for rendering. But if we look at the type of the message we just made, we notice that it's not just a list of products, it's a `WebData (List Product)`. Well, luckily if we [take a look at the type of the Config](http://package.elm-lang.org/packages/splodingsocks/hipstore-ui/latest/HipstoreUI#Config) that we need to pass into HUI, we see that it's expecting what we already have, a `WebData (List Journal)`. So all we have to do is stick the content of the message directly on the model.

Here's the model, and the updated init function with the new field:

```elm
---- MODEL ----


type alias Model =
    { products : WebData (List Product)
    }


init : ( Model, Cmd Msg )
init =
    ( { products = RemoteData.NotAsked
      }
    , Cmd.batch []
    )

...
```

Notice how the initial value of the products field isn't an empty list, but rather `RemoteData.NotAsked`. This just communicates to whomever is reading the data that we haven't sent the request yet.

Now that we've got the field on the model, go ahead and change the update function to handle the new message, and update the model accordingly:

```elm
...

update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        NoOp ->
            model ! []

        ProductsChanged products ->
            { model | products = products } ! []
```

### Actually Sending the Request

So, once we have a command, how do we actually send it to the runtime?

![](https://cdn.meme.am/cache/instances/folder571/250x250/34189571/picard-point-make-it-so-number-one.jpg)

_not quite._

There are two places we can do this. The first is in the init, which is a tuple of model, and a command \(`(Model, Cmd Msg)`\):

![](/assets/import.png)

And the second is quite similar; the update function, which has the same return type. You might notice the fancy `!` in there after `model`:

![](/assets/import2.png)

The `!` is just a function that takes a model on the left, and a list of commands on the right, and bundles them up into the tuple we need. The important thing is that we can use that list to send off commands.

In this case, we'll stick it in the `init` since we want the products to be fetched right away on page load:

```
...

init : ( Model, Cmd Msg )
init =
    ( { products = RemoteData.NotAsked
      }
    , Cmd.batch [ getProducts ] -- <-- This Cmd will get run on app start!
    )

...
```

If you have the development server running and you try to load up the page right now you'll encounter a nice error in the console.

![](/assets/import5.png)

Remember when we defined the body of `decodeProduct` as `Debug.crash "TODO"`? Well the app is doing just that, crashing. Let's learn some JSON decoding and get rid of that crash!

## JSON Decoding

![](http://18404-presscdn-0-57.pagely.netdna-cdn.com/wp-content/uploads/2011/02/hole-in-the-wall-Tom-Riles-Audience-Warm-up-Guy.jpg)

When I first started using Elm I thought JSON decoding was a big pain. I didn't realize that it's Elm's way of keeping the compiler honest, and making sure that the types that come into the app actually are what they claim to be.

Similar to the way in which commands are just data that represent some future action, decoders are data that represent our expectations about the shape of incoming data.

Though the analogy isn't perfect, I'll relate JSON decoding to the U.S. game show ["Hole in the Wall"](https://en.wikipedia.org/wiki/Hole_in_the_Wall_%28U.S._game_show%29) in which contestants attempt to fit themselves through hole cut into a wall moves quickly toward them. If they fit through the hole, they win. If they don't manage to fit themselves through the hole, the wall breaks, and the contestant loses, falling into a poll of water.

Similarly, the incoming JSON is like the participant. If it doesn't fit into the shape we've described with the decoder, the process aborts and we're informed of the error. If the data does fit, we get the data we expected out the other side.

So let's get rid of the Debug.crash from the `decodeProduct` definition and replace it with a decoder. We'll build the decoder using a third-party library, `NoRedInk/elm-decode-pipeline`.

> ~ Aside ~
>
> In Elm, record type aliases are also type constructors, or functions. The type constructor takes as many parameters as it has fields, and in the same order, so:
>
> ```elm
> type alias ABC =
>     {a : String, b: String, c: String}
>
> -- ABC has the type: String -> String -> String -> ABC
> ```

Here's what we'll do:

* Call `decode` with the type constructor \(`Product`\) to wrap up the constructor function in the `Decoder` type.
* One by one, use pipes to apply decoders to that wrapped up constructor function. These other decoders should match up with the types that the constructor takes as parameters.
* Once we've passed in a decoder for each argument in the constructor function, we end up with a `Decoder Product` 

Here we wrap up the constructor function:

```elm
decodeProduct : Json.Decode.Decoder Product
decodeProduct =
    Json.Decode.Pipeline.decode Product
```

And we apply the first decoder, which is a `String`, and will fill the `id` field on the product:

```elm
decodeProduct : Json.Decode.Decoder Product
decodeProduct =
    Json.Decode.Pipeline.decode Product
        |> Json.Decode.Pipeline.required "id" Json.Decode.string
```

Let's take a quick break to explain what's going on here. `Json.Decode.Pipeline.required` is designed to find a named field on a JSON object, and extract the value. It takes a string as the first argument, which is the name of the field on the JSON object, and a decoder as the second argument, which is the type we expect to find there. In this case we're telling it to look for an "id" field on the JSON object, and pull out the string that it finds there. If that field isn't there, or if it isn't a string, the decoding fails, and we know that our expectations about the shape of the data were incorrect.

Now let's do the rest of the fields. We've got to be careful, though. I deliberately designed the API so that the JSON fields don't all match up with the fields we have on the Elm type. **Remember: **when we're decoding, we're not just trying to represent the JSON as-is in Elm, but trying to coerce the JSON into the data types that we've already established in Elm.

Here's the shape of the data that the API returns compared with the fields that HipstoreUI expects:

```
--- API (Typescript)

class Product {
  name: string
  price: number
  image: string
  id: string
}

--- HipstoreUI (Elm)

type alias Product = 
    { id : String!    , displayName : String
    , tacos : Float
    , image : String
    }
```

Looks like the JSON has all of the information we need. But the `displayName` field is called `name` on the JSON, and the `tacos` field is called `price` on the JSON. That's simple enough to handle. All we need to do is use the same `required` function we used above, but make sure we use the right name for the JSON field:

```
decodeProduct : Json.Decode.Decoder Product
decodeProduct =
    Json.Decode.Pipeline.decode Product
        |> Json.Decode.Pipeline.required "id" Json.Decode.string
        |> Json.Decode.Pipeline.required "name" Json.Decode.string
         -- 👆 This is the second application, so it corresponds with
         -- the displayName field on the Product record.
```

You've got it! You're doing very well. I'll leave you to use what you learned above to apply decoders to the last two fields, tacos and image. Good luck

## 

## Sending the HTTP Request

* Cmds
* Msgs
* Update function

## Rendering the Page

## Bonus Points

**TODO: **There's already a loading indicator provided by HipstoreUI that animates a loading bar whenever an XHR is in progress. This can be disabled by setting `loadingIndicator` to false on the config record you're passing to the HipstoreUI function. Can you implement your own loading indicator that displays when the request is fired off, and hides when the request returns?

