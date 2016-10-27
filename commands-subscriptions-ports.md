# Elm: Commands and Subscriptions and Ports. Oh My!

[Two weeks ago](https://steemit.com/elm/@billstclair/elm-rocks), I waxed damn near poetic about the [Elm programming language](http://elm-lang.org). Since then, I've made fully functional my first Elm webapp, [Kakuro-Dojo](http://kakuro-dojo.com/). I still have plenty more features to add, but it's playable, and lays out nicely on mobile (well, at least on my iDevices).

<center>
![Elm Logo](https://billstclair.com/steemit/elm-rocks/elm-logo.png)<br/>Logo Credit: [@elmlang](https://twitter.com/elmlang)
</center>

In this article, I'm going to attempt to explain three things that I found difficult to understand: commands, subscriptions, and ports.

## Commands and Subscriptions

We'll start with commands and subscriptions, and then add ports.

First, download [some code](https://github.com/billstclair/elm-csp) from GitHub, and aim ```elm-reactor``` at it:

```
git clone https://github.com/billstclair/elm-csp.git
cd elm-csp
elm reactor
```

Aim your web browser at http://localhost:8000, and click on "```csp.elm```" (NOT "```csp2.elm```"). You should see a big random number between 1 and 100 that changes every second and a "Time" counter that increments every second. If you refresh the browser, the time will return to 0.

This code basically puts together the [Random](http://elm-lang.org/examples/random) and [Time](http://elm-lang.org/examples/time) examples on the Elm  [Learn by Example](http://elm-lang.org/examples) page. They're explained in some detail in the [Get Started](http://elm-lang.org/get-started) book. But, though those examples work, for some reason I didn't [grok](https://en.wikipedia.org/wiki/Grok) commands and subscriptions right away.

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

[Time.every](http://package.elm-lang.org/packages/elm-lang/core/4.0.5/Time#every) takes a ```Time``` (number of millisecond ticks) as first arg and a function from ```Time``` to ```msg```, ```(Time -> msg)```, as second arg. It returns a subscription to that same ```msg``` type, ```Sub msg```. I use ```Tick``` as the function from ```Time``` to ```msg```, it returns not just a generic ```msg``` type, but an instance of ```Msg```, specifically, ```Tick Time```.

The ```subscriptions``` function gets a Model as input, but in this example, it ignores that.

Bottom line, this causes ```Tick``` to be called once a second with an argument of the current Time, in milliseconds (but use Time.milliseconds, Time.seconds, and friends to insulate your code from clock precision). ```Tick``` produces a ```Tick Time``` ```Msg```, which is passed by the Elm runtime to to the ```update``` function.

You won't always want to use one of your ```Msg``` functions directly. For example, since our ```update``` function doesn't use the time, we could have defined ```Tick``` without an argument, and ignored the time argument when we created that message:

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

Now that we have the Elm runtime delivering us regular Tick messages, what do we do with them:

```
update msg model =
  case msg of
    Tick newTime ->
      ({ model | time = model.time + 1 }
       , Random.generate NewRandom <| Random.int 1 100
       )

    NewRandom r ->
      ( { model | random = r }, Cmd.none)
```

```Random``` is an effects module. It keeps its own internal random seed. You can keep track of the seed yourself if you want, and then you wouldn't need the command to pass through the effects module and come back, but I'm documenting commands here, so that's what I did.

```Random.generate``` takes as first arg a function that maps an ```Int``` to a ```msg```. We're passing it the ```NewRandom``` function. Again, we're using the type function itself. But it could be any function that takes an integer as input and generates a NewRandom message. I could have coded it as:

```
makeNewRandom : Int -> Msg
makeNewRandom r =
  NewRandom r

update msg model =
  case msg of
    Tick newTime ->
      ({ model | time = model.time + 1 }
       , Random.generate makeNewRandom <| Random.int 1 100
       )
    ...
```

The second arg to ```Random.generate``` is a ```Random.Generator```. I'm passing ```Random.int 1 100``` for that, which generates a random integer between 1 and 100.

```Random.generate``` returns a command. We don't have to care about which type of command, we just return it as the second element of the ```(Model, Cmd)``` tuple, and the runtime will pass it to the ```Random``` effects code, which will generate the random number and pass the resulting ```Msg``` back to our update function. When we receive it, we just stuff it into the ```Model```, and we're done:

```
    NewRandom r ->
      ( { model | random = r }, Cmd.none)
```

## Ports

```csp2.elm``` adds ports to ```csp.elm```. Since the code that implements a port needs to be coded in JavaScript, you need to have an HTML file to contain the JavaScript code. ```index.html``` is that HTML file. You can't run a port module in ```elm-reactor```. In order to run it, you need to compile the elm into JavaScript, and refernce that JavaScript in the HTML file:

```
cd .../elm-csp
elm make csp2.elm --output csp2.js
```

Then you can open ```index.html``` in your browser.

The following line in the &lt;head> section of ```index.html``` is where the Elm code gets included:

```
  <script type="text/javascript" src="csp2.js"></script>
```

I copied my port code from Evan Czaplicki's [TodoMVC example](https://github.com/evancz/elm-todomvc). You can read more about JavaScript interop with ports [here](https://guide.elm-lang.org/interop/javascript.html).

In order to have ports in a module, you have to declare it to be a port module:

```
port module Csp exposing (..)
```

That requires you to use ```App.programWithFlags```, so that your ```init``` function will get an argument from the JavaScript code:

```
main =
  App.programWithFlags
    { init = init
    , view = view
    , update = updateWithStorage
    , subscriptions = subscriptions
    }

...

init : Maybe Model -> (Model, Cmd Msg)
init maybeModel =
  let model = case maybeModel of
                   Nothing ->
                     { version = modelVersion
                     , random = 0
                     , time = 0
                     , name = ""
                     }
                   Just m ->
                     m
  in
      (model, Cmd.none)
```

The first arg of your ```init``` function can be of any type you want, as long as the JavaScript code passes the JavaScript version of that type. If you put ```Maybe``` in front of the type, then the JavaScript code can pass ```null``` to denote ```Nothing```. From ```index.html```:

```
// Must match modelVersion in csp2.elm
var modelVersion = 2;

var name = '';
var storedState = localStorage.getItem(storageName(name));
var startingState = storedState ? JSON.parse(storedState) : null;
if (startingState && startingState.version != modelVersion) {
  startingState = null;
}
var csp = Elm.Csp.fullscreen(startingState);
```

```localStorage``` is a simple key-value store implemented by modern browsers that enables caching data in persistent storage. It's implemented as an SQLite database file for each domain that uses it in my macOS browsers. [Read about it here](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage).

The save code in ```index.html``` simply converts what it gets to and from a JSON-encoded string, with one exception. Because an application's saved Model tends to change, it's good to have a ```version``` field in that record, so that you can recognize old saved state and either throw it out, which is what my code does above, or convert it to the new version, which is what you should do in a real, production-quality application (see the note at the end of this article).

This brings up an issue of how Elm data structures are represented in JavaScript. From [that Interop page](https://guide.elm-lang.org/interop/javascript.html):

* Booleans and Strings – both exist in Elm and JS!
* Numbers – Elm ints and floats correspond to JS numbers
* Lists – correspond to JS arrays
* Arrays – correspond to JS arrays
* Tuples – correspond to fixed-length, mixed-type JS arrays
* Records – correspond to JavaScript objects
* Maybes – Nothing and Just 42 correspond to null and 42 in JS
* Json – Json.Encode.Value corresponds to arbitrary JSON

The only other issue is how the JavaScript starts up the Elm code:

```
var csp = Elm.Csp.fullscreen(startingState);
```

```Elm``` is defined by the JavaScript code generated by the compiler in ```csp2.js```. ```Csp``` is the name of the module:

```
port module Csp exposing (..)
```

```fullscreen``` is the name of the function that starts up your Elm code with control of the whole screen and passes ```startingState``` to the ```init``` function. If you want to code some of your window's appearance in the HTML file, you can call Elm.Csp.embed. Search for ".embed" on [that interop page](https://guide.elm-lang.org/interop/javascript.html) for details. The Elm architecture ensures that data coming back into Elm from JavaScript always satisfies type constraints. If your JavaScript code is faulty, you'll get JavaScript errors, meaning you'll become friends with the "Debuger Tools" in your browser while working on that small part of your application, but once the data gets back into Elm, you're once again in error-free territory.

On to the ports themselves. An individual port can be used for output to the JavaScript or for input from the JavaScript, but not both. Sometimes they come in pairs. My application has one output port and one input/output pair. Output ports are functions that return a command. They are created in the ```update``` function. Input ports are functions that return a subscription. They are part of the application's ```subscribe``` function. Here are the three ports in my example:

```
-- save (name, model)
port save : (String, Model) -> Cmd msg

-- request name
port request : String -> Cmd msg
port receive : (Maybe Model -> msg) -> Sub msg
```

The ```save``` port is output only. It is used to save the game state. Its first argument is the name of the saved game, which is typed into the text box in the UI. Its second argument is the Model. The save is initiated by the Save message, created by the "Save" button:

```
update : Msg -> Model -> (Model, Cmd Msg)
update msg model =
  case msg of
    ...
    Save ->
      ( model, save ( model.name, model ))
    ...
```

A save to the blank name (""), which is the one restored when the page loads, happens automatically in the ```updateWithStorage``` function, which is the actual "update" function for the application:

```
main =
  App.programWithFlags
    { init = init
    , view = view
    , update = updateWithStorage
    , subscriptions = subscriptions
    }

updateWithStorage : Msg -> Model -> ( Model, Cmd Msg )
updateWithStorage msg model =
  let
    ( newModel, cmds ) = update msg model
  in
      ( newModel
      , Cmd.batch [ save ("", newModel), cmds ]
      )
```

[```Cmd.batch``` changes a list of commands into a single batch command.]

The ```save``` port invokes the following JavaScript in ```index.html```. Note how it pulls ```name``` and ```state``` from the Elm ```(name, state)``` tuple:

```
csp.ports.save.subscribe(function(pair) {
  var name = pair[0];
  var state = pair[1];
  var json = JSON.stringify(state);
  localStorage.setItem(storageName(name), json);
});
```

Finally, in order to read a saved state, the "Read" button invokes the ```Restore``` message, which call the ```request``` port to create a command:

```
update : Msg -> Model -> (Model, Cmd Msg)
update msg model =
  case msg of
    ...
    Restore ->
      ( model, request model.name )
    ...
```

That invokes the following JavaScript in ```index.html```, which reads the JSON string from the database, parses it, ensures that its version is correct, and sends it back to the Elm code through the ```receive``` port:

```
csp.ports.request.subscribe(function(name) {
  var json = localStorage.getItem(storageName(name));
  var state = json ? JSON.parse(json) : null;
  if (state && state.version != modelVersion) {
    state = null;
  }
  csp.ports.receive.send(state);
});
```

We subscribe to those ```receive```s in the modified ```subscriptions``` function:

```
subscriptions : Model -> Sub Msg
subscriptions model =
  Sub.batch [ Time.every second Tick
            , receive Receive           -- ***HERE'S THE SUBSCRIPTION***
            ]
```

And we process the ```Receive``` message in the ```update``` function:

```
update : Msg -> Model -> (Model, Cmd Msg)
update msg model =
  case msg of
    ...
    Receive maybeModel ->
      let model' = case maybeModel of
                       Nothing -> model
                       Just m -> m
      in
          ( model', Cmd.none)
```

## Conclusion

I hope that this article makes it a little easier for you than it was for me to grok commands, subscriptions, and ports.

If you need to send some data out of your Elm code to somewhere you can't get in normal Elm code, you're probably going to use a command.

If you need to receive some data from outside, you're probably going to use a subscription.

One of the places you can send out data with a command or bring it in with a subscription is JavaScript code. You do that with a port.

Bill St. Clair &lt;[billstclair@gmail.com](mailto:billstclair@gmail.com)&gt;<br/>26 October 2016

------

Note that it's generally considered bad practice to rely on the compiler-generated conversion code for converting your ```port``` parameters to and from JavaScript. It's better to pass only strings, and to explicitly convert Elm data structures to and from JSON strings. This makes your job much easier if you have to do schema update when you change the form of the saved data. And you'll also avoid a [compiler bug](https://github.com/elm-lang/elm-compiler/issues/1510), possibly fixed by now, but there when I wrote this on 26 October 2016, that causes a run-time error in the auto-generated conversion code if any of your record fields is named "default".
