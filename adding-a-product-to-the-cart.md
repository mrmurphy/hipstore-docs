# Adding a Product to the Cart

Adding a product to the cart will work just like the previous HTTP request except, instead of using a GET request, we'll use a POST. 

## Constructing a POST Request

There's a function in the `Http` module called `post` that makes building simple POST requests easy. Unfortunately, for this request we need the cookie, and so we'll have to build the request in the same way that we built the `getCart` request:

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

Now, on your own, go ahead and add a new Msg that will trigger this request, and add that message to the `onAddToCart` field in the config we're passing to HUI.

![](/assets/import7.png)

Party! Items in the cart.



