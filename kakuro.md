# Kakuro

[Kakuro](http://www.kakuro.com/) is a cross-sums puzzle game, sort of a crossword puzzle with numbers. Each horizontal or vertical "clue" is the sum of the 1 to 9 numbers in its solution squares. Each solution square holds a unique number from 1 to 9 (no repeated numbers). I find it more interesting than Sudoku. It's not as frenetic, and requires more logic.

I've been using the [Real Kakuro](https://itunes.apple.com/us/app/real-kakuro/id459669325) app on my iPhone. Extremely annoying ads, possibly worth the $3 to make them go away. Helpful hints. I find its "Easy" puzzles easy, "Medium" not too hard, "Hard" doable, "Challenging" possible but time-consuming, and I haven't yet tried an "Extreme" puzzle.

[KakuroConquest.com](http://www.kakuroconquest.com/) offers free 8x8 and 9x11 online JavaScript puzzles, thousands of them. Here's one of their easy puzzles, partially filled in:

<center>
![easy 8x8 KakuroConquest.com puzzle](https://billstclair.com/steemit/kakuro/kakuro-conquest.png)
</center>

This article contains an outline of what I've learned about solving Kakuro, and some ideas for an easier-to-use app, which I may write in Swift for iPhone or JavaScript for the web, assuming I can figure out how to automatically generate puzzles with the desired difficulty level.

## Language

`[sum,count]` denotes a clue for `count` squares that add to `sum`.

`(x,y,z)` denotes a solution with three numbers: `x`, `y`, & `z`. Multiple solutions are separated by semicolons.

`[sum,3] = (x,y,z);(a,b,c)` denotes two possible three-number solutions.

## Single solutions

The following clues have only a single solution.

```
[3,2]   = (1,2)
[4,2]   = (1,3)
[16,2]  = (7,9)
[17,2]  = (8,9)
[6,3]   = (1,2,3)
[7,3]   = (1,2,4)
[23,3]  = (6,8,9)
[24,3]  = (7,8,9)
[10,4]  = (1,2,3,4)
[11,4]  = (1,2,3,5)
[29,4]  = (5,7,8,9)
[30,4]  = (6,7,8,9)
[15,5]  = (1,2,3,4,5)
[16,5]  = (1,2,3,4,6)
[34,5]  = (4,6,7,8,9)
[35,5]  = (5,6,7,8,9)
```

## Two solutions

The following clues have two solutions.

```
[5,2]   = (1,4);(2,3)
[6,2]   = (1,5);(2,4)
[14,2]  = (5,9);(6,8)
[15,2]  = (6,9);(7,8)
[8,3]   = (1,2,5);(1,3,4)
[9,3]   = (1,2,6);(1,3,5)
[21,3]  = (4,8,9);(5,7,9)
[22,3]  = (5,8,9);(6,7,9)
```

## Techniques

Shamelessly stolen [from Kakuro.com](http://www.kakuro.com/techniques.php).

### Lone square

If only one square is missing from a run, do the math.

```
9| 4 X 3
```

4 + 3 = 7. 9 - 7 = 2.

```
9| 4 2 3
```

### Cross reference

If two runs cross, and share only a single number in their solutions, that number goes in the shared square.

```
   17
   -- --
  | A
16| X  B
```

Since `[17,2]=(8,9)` and `[16,2]=(7,9)`, `X` below must be `9`, hence `A` is `8` ,and `B` is `7`.

```
   17
   -- --
  | 8
16| 9  7
```

### Combo reference

If two runs cross a third, it is sometimes possible to solve for the third run.

```
 | 4  3
  -- --
 | A  B
7| X  Y  C
```

`[4,2]=(1,3)`, `[3,2]=(1,2)`, `[7,3]=(1,2,4)`. Since `4` isn't in the sums for `4` or `3`, `C` must be `4`. `X` cant be `2`, so it must be `1`. That means `Y` is `2`, `A` is `3`, and `B` is `1`.

```
   4  3
  -- --
 | 3  1
7| 1  2  4
```

### More

Those three will get you a lot. I'll leave you to read about two more techniques on [that Kakuro.com page](http://www.kakuro.com/techniques.php).

## My app ideas

The Real Kakuro app offers possible solutions for the clues in the row and column of the selection, filtering by the already filled-in squares. This helps a lot, but it would be more useful to highlight the numbers that are not yet filled in.

Real Kakuro also provides a pencil tool, that allows you to make notes on empty squares. But those notes do not filter the possible solutions, so you have to do that yourself.

I want to have colored trials. Committed numbers would be in black. Trial one would be blue, trial two in green, trial three in orange. With selectable color schemes to accomodate the color blind. Each trial would have a set of initial numbers. There would be commands to erase that trial except for the initial numbers, so you could change the initial guess, to merge a trial into the higher-level trial (or the solution), and to erase a whole trial.
