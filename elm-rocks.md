# Elm Rocks!

I reported two months back on starting a JavaScript app called [Kakuro Master](https://steemit.com/kakuro/@billstclair/kakuro-master). I worked on it for a little while, but discovered that generating new board layouts is hard, so I lost interest. Well, my interest has returned, due to learning a new programming language, [Elm](http://elm-lang.org).

<center>
![Elm Logo](https://billstclair.com/steemit/elm-rocks/elm-logo.png)
Logo Credit: [@elmlang](https://twitter.com/elmlang)
</center>

Elm is basically a simplified [Haskell](https://www.haskell.org), that compiles into JavaScript, and has a nice run-time that makes it easy to use it to create webapps. The apps can be embedded inside an existing web page, or you can give Elm the whole screen, and write no HTML, CSS, or JavaScript, except via Elm libraries that generate it.

When I started looking around the Elm code on GitHub, I discovered that the Elm compiler is written in Haskell. I didn't recognize the similarity until then, having very little experience with Haskell. So I went back to a tutorial that I had started a while back: [_Learn You a Haskell for Great Good!_](http://learnyouahaskell.com/) I learned enough to write [my first Haskell program](https://lisplog.org/my_first_haskell_program.html), and I was hooked. I've been thinking my next programming language would be [Elixir](http://elixir-lang.org)/[Erlang](http://www.erlang.org/), but I think now that it will be Haskell on the server and Elm on the client, though a lot depends on who offers to pay me for playing ([Clojure](http://clojure.org) also doesn't suck, though I'm not happy about its dependence on Java).

Elm and Haskell (and Elixir and Erlang) are a bit weird for most programmers. They are all pure functional languages. All data is read-only. You can create new versions of data structures that share lots of components, but there is never an issue of locking around state changes, because values are immutable. There are a few exceptions to this, since they're necessary for interaction between processes and for I/O, but they're limited, and designed to make it easy to avoid deadlock.

Haskell is also lazy. Nothing is computed until you ask for it. You can create lists of infinite length, but unless you try to print one, or map over the whole thing, it will never actually be created. Very cool.

I'm not going to bother with examples. If you're interested, take a gander at [_Learn You a Haskell..._](http://learnyouahaskell.com/), or work through the <a href='https://guide.elm-lang.org/get_started.html'>Elm tutorial</a>.

So I've started converting [Kakuro Master](http://kakuro-master.com) to Elm. I have the basic interface looking pretty much as it used to. Now I need to go back to the board generation code. Once I get that done, I'll write the code to display the board with sums for rows and columns, instead of the debugging output it has now, which shows the numbers in each cell. Then the game play will be easy. And I'll finally have the new feature I talked about in my [earlier article](https://steemit.com/kakuro/@billstclair/kakuro-master).

The new code isn't yet live at [kakuro-master.com](http://kakuro-master.com/), but as soon as I get something to generate new board layouts, I'll switch to the new Elm version, and put "Elm inside", at the bottom of the page, with a small version of the logo above.

If you want to look at my code in progress, it's at [github.com/billstclair/kakuro-master](https://github.com/billstclair/kakuro-master/). And [I tweet](https://twitter.com/billstclair) about it tagged with [@myElmStatus](https://twitter.com/myElmStatus).

Elm Rocks!
