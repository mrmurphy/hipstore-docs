# Getting the Cart Contents

Getting the cart contents will work just like the last HTTP request except because of Web security we're going to have to construct the request a bit differently. We're using a session cookie on the server to keep track of what each user has in their cart, and since the server is running on a different domain from the client, we'll need to use the `withCredentials` option on the browser's XMLHttpRequest API to expose that cookie to the server. The built-in `get` function doesn't allow us to do this, but luckily, there's a lower-level function that does.

```elm
getCart : Cmd Msg
getCart =
    Http.request
        { method = "GET"
        , headers = []
        , url = "https://hipstore.now.sh/api/cart"
        , body = Http.emptyBody
        , expect = Http.expectJson (Json.Decode.list decodeProduct)
        , timeout = Nothing
        , withCredentials = True
        }
        |> RemoteData.sendRequest
        |> Cmd.map CartChanged
```

Here we've called `Http.request` and we've passed in a record with all of the fields that function requires. The fields we really care about are `method`, `url`, `expect`, and `withCredentials`. The rest are just given default values to make the compiler happy.

Now, on your own, add a field on the model for the cart contents, and add a message called `CartChanged` to set that field.

Once you've finished, you can hook up the `cart` field from the model to the rendering function we're using from HipstoreUI and it'll display a count of the items in your cart.

Great work! 💪

