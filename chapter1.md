# Bootstrapping

## Hipstore-UI

![](https://thumbs.gfycat.com/SlushyMixedArizonaalligatorlizard-size_restricted.gif)

Even though this is just a workshop, I didn't want the example project to look like the spread of a 1997 personal homepage on the information superhighway, so I pre-baked the user interface and published as an Elm package that exposes a type and two functions–one that renders each page of the app when given the right info. Through the rest of this documentation, that package will be referred to as Hipstore-UI, or HUI. The template comes with it pre-installed, so you shouldn't have to worry about anything there. But it'd be very good to take a look at the documentation so you know what kinds of data and messages we need:

Why don't you take a moment to do that right now?

[http://package.elm-lang.org/packages/splodingsocks/hipstore-ui/3.0.0/HipstoreUI](http://package.elm-lang.org/packages/splodingsocks/hipstore-ui/3.0.0/HipstoreUI)

## The Template

![](https://media0.giphy.com/media/Qrjc3clJyAQU0/giphy.gif?response_id=5920e3ada1417066159ae980)

In order to make the most of our short time together, I've taken the liberty of setting up a starter template that has all of the packages we need pre-installed, and has a skeleton of an app already set up. Basically it just has enough to get a placeholder up on screen. That project can be found here:

[https://github.com/splodingsocks/hipstore-starter](https://github.com/splodingsocks/hipstore-starter)

Please take a moment to clone that repository and follow the installation instructions.

The template has all of its code in one single file. In real-world situations, you'd want to split that monolith of a file up into multiple modules for better scalability. But modules aren't the focus of this workshop, so we're going to favor the simplicity of the single file approach \(even at the risk of exhausting your scroll-wheel\).

There are a couple of things worth noting in the template. First, the model exists, but it's empty:

```
---- MODEL ----


type alias Model =
    {}

...
```

You'll be in charge of adding new fields onto that model as necessary throughout the workshop.

Next, there's only one `Msg` right now, and that message does nothing at all:

```
---- UPDATE ----


type Msg
    = NoOp


update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        NoOp ->
            model ! []
```

Once again, the Msg type is ready for you to fill up with new constructors at your leisure.

Lastly, there's already a view function that uses HUI:

```
---- VIEW ----


uiConfig : Model -> HipstoreUI.Config Msg
uiConfig model =
    { onAddToCart = \_ -> NoOp
    , onRemoveFromCart = \_ -> NoOp
    , onClickViewCart = NoOp
    , onClickViewProducts = NoOp
    , products = RemoteData.NotAsked
    , cart = RemoteData.NotAsked
    , loadingIndicator = True
    }


view : Model -> Html Msg
view model =
    div []
        [ HipstoreUI.products <| uiConfig model
        ]
```

Notice that the uiConfig matches up with the type required by HUI for both the products and the cart functions. Right now it's filled up with default values for `products`, `cart`, and `loadingIndicator`, and all of the parts that take messages are just using `NoOp`. It'll be your job to fill these fields in with real values by the end of the workshop.

## The API

I've prepared a simple API to code against. It might be a bit slow in response because it's hosted on the free tier of [https://zeit.co/now](https://zeit.co/now). Please take a moment to read the simple API documentation at [https://hipstore.now.sh](https://hipstore.now.sh).

## The Environment

The starter kit has detailed instructions about what you need to get going, but it's really just three things: `Node.js`, `elm`, and `create-elm-app`. See more in the readme.

