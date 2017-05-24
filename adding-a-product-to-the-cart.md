# Adding a Product to the Cart

Adding a product to the cart will work just like the last two HTTP requests except, instead of using a GET request, we'll use a POST. The construction of a POST request is a bit different from the construction of a GET, so we'll go over that here. The rest will be left to you since you're already a pro at this 😎.

## Constructing a POST Request

There's a function in the `Http` module called `post` that makes building simple POST requests easy. Unfortunately, because of the way our server is set up, and the way browser security works, we're going to have to use the `withCredentials` option on the browser's XMLHttpRequest API. Luckily, the `Http.request` function gives us access to that:

```elm
addItemToCart : String -> Cmd Msg
addItemToCart id =
    Http.request
        { method = "POST"
        , headers = []
        , url = "https://hipstore.now.sh/api/cart/" ++ id
        , body = Http.emptyBody
        , expect = Http.expectJson (Json.Decode.list decodeProduct)
        , timeout = Nothing
        , withCredentials = True
        }
        |> RemoteData.sendRequest
        |> Cmd.map CartChanged
```

Here we've called `Http.request` and we've passed in a record with all of the fields that function requires. The fields we really care about are `method`, `url`, `expect`, and `withCredentials`. The rest are just given default values to make the compiler happy.

Now, on your own, go ahead and add a new Msg that will trigger this request, and add that message to the `onAddToCart` field in the config we're passing to HUI.

