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

### Making a place on the model for the data that we're expecting the request to return

### Hand the command off to the runtime

## JSON Decoding

## Sending the HTTP Request

* Cmds
* Msgs
* Update function

## Rendering the Page

## Bonus Points

**TODO: **There's already a loading indicator provided by HipstoreUI that animates a loading bar whenever an XHR is in progress. This can be disabled by setting `loadingIndicator` to false on the config record you're passing to the HipstoreUI function. Can you implement your own loading indicator that displays when the request is fired off, and hides when the request returns?

