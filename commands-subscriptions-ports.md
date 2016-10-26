# Elm: Commands and Subscriptions and Ports. Oh My!

[Two weeks ago](https://steemit.com/elm/@billstclair/elm-rocks), I waxed damn near poetic about the [Elm programming language](http://elm-lang.org). Since then, I've made fully functional my first Elm webapp, [Kakuro-Dojo](http://kakuro-dojo.com/). I still have plenty more features to add, but it's playable, and lays out nicely on mobile (well, at least on my iDevices).

<center>
![Elm Logo](https://billstclair.com/steemit/elm-rocks/elm-logo.png)
Logo Credit: [@elmlang](https://twitter.com/elmlang)
</center>

In this article, I'm going to attempt to explain three things that I found difficult to understand: commands, subscriptions, and ports.

## Commands and Subscriptions

We'll start with commands and subscriptions, and then add ports.

First, download some code from GitHub, and aim ```elm-reactor``` at it:

```
git clone https://github.com/billstclair/elm-csp.git
cd elm-csp
elm reactor
```

Aim your web browser at http://localhost:8000, and click on "```csp.elm```" (NOT "```csp2.elm```"). You should see a big random number between 1 and 100 that changes every second and a "Time" counter that increments every second. If you refresh the browser, the time will return to 0.

This code basically puts together the [Random](http://elm-lang.org/examples/random) and [Time](http://elm-lang.org/examples/time) examples on the Elm  [Learn by Example](http://elm-lang.org/examples) page. They're explained in some detail in the (Get Started)[http://elm-lang.org/get-started] book. But, though those examples work, for some reason I didn't grok command and subscriptions right away.

The important parts of this example are the ```Msg``` type and the ```update``` and ```subscriptions``` functions:

```
-- UPDATE

type Msg
  = Tick Time
  | NewRandom Int

update : Msg -> Model -> (Model, Cmd Msg)
update msg model =
  case msg of
    Tick newTime ->
      ({ model | time = model.time + 1 }
       , Random.generate NewRandom <| Random.int 1 100
       )

    NewRandom r ->
      ( { model | random = r }, Cmd.none)

-- SUBSCRIPTIONS

subscriptions : Model -> Sub Msg
subscriptions model =
  Time.every second Tick
```

## Subscriptions

[Time.every](http://package.elm-lang.org/packages/elm-lang/core/4.0.5/Time#every) takes a ```Time``` (number of millisecond ticks) as first arg and a function from ```Time``` to ```msg```, ```(Time -> msg)```, as second arg. It returns a subscription to that same ```msg``` type, ```Sub Msg```. I use ```Tick``` as the function from ```Time``` to ```msg```, it returns not just a generic ```msg``` type, but an instance of ```Msg```, specifically, ```Tick Time```.

The ```subscriptions``` function gets a Model as input, but in this example, it ignores that.

Bottom line, this causes ```Tick``` to be called once a second with an argument of the current Time, in milliseconds (but use Time.milliseconds, Time.seconds, and friends to insulate your code from clock precision). ```Tick``` produces a ```Tick Time``` ```Msg```, which is passed by the Elm runtime to to the ```update``` function.

You won't always want to use one of your ```Msg``` functions directly. For example, since our ```update``` function doesn't use the time, we could have defined ```Tick``` without an argument, and ignored the time argument when we generated that message:

```
Type Msg
  = Tick
  | NewRandom Int
  
subscriptions : Model -> Sub Msg
subscriptions model =
  Time.every second (\time -> Tick)
```

(You may need a refresher on [anonymous functions](https://www.elm-tutorial.org/en/01-foundations/02-functions.html))

## Commands

