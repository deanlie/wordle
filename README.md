
<!-- README.md is generated from README.Rmd. Please edit that file -->

# wordle

<!-- badges: start -->

![](https://img.shields.io/badge/cool-useless-green.svg)
[![R-CMD-check](https://github.com/coolbutuseless/wordle/workflows/R-CMD-check/badge.svg)](https://github.com/coolbutuseless/wordle/actions)
<!-- badges: end -->

The `{wordle}` package contains code to assist in finding good candidate
words for Wordle.

“Wordle” itself is a guess-a-word puzzle [playable
online](https://www.powerlanguage.co.uk/wordle/).

The game plays like the old ‘mastermind’ board game, but with letters
instead of coloured pins. The gameplay is as follows:

1.  Enter a word as a guess for the hidden target word.
2.  Any letters which are within the hidden target word are coloured in
    yellow.
3.  Any letters which match exactly the letter in the hidden target word
    are coloured green
4.  Figure out a new candidate word as a guess for the hidden target
    word, and go back to Step 1.

In the following game of Wordle, the first guess was `eaten`, the second
was `arise`, and then the third guess really only has one good option
given the constraints revealed so far: `aside`. This was the hidden
target word, which means the puzzle is solved!

<img src="man/figures/eg.png" />

The process of finding good candidate words given letters which have
been seen so far is a good match for regular expressions. This package
aims to help you find these good candidate words.

## What’s in the box

-   `play_wordle()` Simple way to play a game of wordle in the R console
-   `wordle_dict` an ‘official’ list of words scraped from the Wordle
    website
-   `WordleHelper` R6 Class is the primary way of finding candidate
    words. It has the following methods:
    -   `$new()` to start a new object to help with a new puzzle.
    -   `$words` to get a list of all the remaining possible valid words
        given the words and responses that have been seen so far
    -   `$update()` to notify the object of what the latest `word` was,
        and the colour responses received back from the game for each
        letter.
    -   The `WordleHelper` class is a stateful wrapper around
        `filter_words()`

Advanced:

-   `letter_freq` order of letters by frequency in the `wordle_dict`
-   `orthogonal_words` sets of `n` words which encompass the top `n*5`
    letters (by word frequency in the `wordle_dict`)
-   `filter_words()` is a stateless function for filtering a list of
    words by various constraints on letter position.
-   `WordleGame` R6 class representing a wordle game engine
    -   `$new()` to create a new game and internally choose a target
        word for this game
    -   `$try(word)` to try a word and see what the response is in
        relation to the hidden target word
    -   `$share()` create a block of unicode representing the evolution
        of the solution process.
    -   `play_wordle()` is a simple helper function wrapping this class.

## Installation

You can install from [GitHub](https://github.com/coolbutuseless/wordle)
with:

``` r
# install.packages('remotes')
remotes::install_github('coolbutuseless/wordle')
```

# Play a game of Wordle in your R console

``` r
wordle::play_wordle()
```

<img src="man/figures/game.png" />

# Help solve a puzzle with `wordle::WordleHelper`

In this example, after picking my favourite starting word, at each step
I will just pick the first word in the alphabetical list of remaining
possible words.

<img src="man/figures/00.png" />

``` r
helper <- WordleHelper$new(nchar = 5)
length(helper$words)
#> [1] 12972
head(helper$words)
#> [1] "aahed" "aalii" "aargh" "aarti" "abaca" "abaci"
```

## Initial word choice: `arose`

There are many opinions on a good starting word - I like: `arose`

<img src="man/figures/01.png" />

Update puzzle state with the word played and the response:

``` r
helper$update("arose", c('grey', 'grey', 'grey', 'yellow', 'green'))
helper$words
#>  [1] "besee" "disme" "ensue" "esile" "fusee" "geste" "gusle" "issue" "istle"
#> [10] "lisle" "mesne" "piste" "pusle" "scene" "scute" "sedge" "segue" "seine"
#> [19] "seize" "selle" "semee" "semie" "sente" "shine" "shite" "shive" "shule"
#> [28] "shute" "sidhe" "sidle" "siege" "sieve" "since" "singe" "sithe" "sixte"
#> [37] "skene" "skite" "skive" "skyte" "slice" "slide" "slime" "slipe" "slive"
#> [46] "slype" "smeke" "smile" "smite" "snide" "snipe" "spice" "spide" "spike"
#> [55] "spile" "spine" "spite" "spule" "spume" "stede" "stele" "steme" "stile"
#> [64] "stime" "stipe" "stive" "stude" "stupe" "style" "styme" "styte" "suede"
#> [73] "suete" "suite" "sujee" "swede" "swile" "swine" "swipe" "swive" "sybbe"
#> [82] "sycee" "sythe" "teste" "unsee" "upsee" "usque" "visie" "visne"
```

## Choose the first word: `besee`

<img src="man/figures/02.png" />

Update puzzle state with the word played and the response:

``` r
helper$update("besee", c('grey', 'yellow', 'yellow', 'grey', 'green'))
helper$words
#>  [1] "esile" "scene" "siege" "sieve" "skene" "smeke" "stede" "stele" "steme"
#> [10] "suede" "suete" "sujee" "swede" "sycee"
```

## Choose the first word: `esile`

<img src="man/figures/03.png" />

Update puzzle state with the word played and the response:

``` r
helper$update("esile", c('yellow', 'yellow', 'yellow', 'grey', 'green'))
helper$words
#> [1] "siege" "sieve"
```

## Choose the first word: `siege`

<img src="man/figures/04.png" />

**Success!**

# Orthogonal Word Sets

`orthogonal_words` are multiple lists of words from 1 to 5 words in a
row. All words are drawn from `wordle_dict`.

Within each set of words there are no duplicated letters.

Within each set of words, the most common N letters from the wordle
dictionary are represented.

E.g. The first 15 most common letters in the wordle dictionary are
`c("s", "e", "a", "o", "r", "i", "l", "t", "n", "u", "d", "y", "c", "p")`.
All the 3-word sets use each of these letters once (and once only) - no
duplicated letters are allowed.

``` r
letter_freq[1:5]
#> [1] "s" "e" "a" "o" "r"
head(orthogonal_words[[1]])  
#>   word1
#> 1 soare
#> 2 aeros
#> 3 arose

letter_freq[1:10]
#>  [1] "s" "e" "a" "o" "r" "i" "l" "t" "n" "u"
head(orthogonal_words[[2]])  
#>   word1 word2
#> 1 sonar tuile
#> 2 suite loran
#> 3 sorel tuina
#> 4 roans tuile
#> 5 lores tuina
#> 6 soler tuina

letter_freq[1:15]
#>  [1] "s" "e" "a" "o" "r" "i" "l" "t" "n" "u" "d" "y" "c" "p" "m"
head(orthogonal_words[[3]])  
#>   word1 word2 word3
#> 1 carts poind muley
#> 2 coats pined murly
#> 3 cared moils punty
#> 4 cared ponty muils
#> 5 mares poind culty
#> 6 cures poind malty

letter_freq[1:20]
#>  [1] "s" "e" "a" "o" "r" "i" "l" "t" "n" "u" "d" "y" "c" "p" "m" "h" "g" "b" "k"
#> [20] "f"
head(orthogonal_words[[4]])
#>   word1 word2 word3 word4
#> 1 barks child pongy fumet
#> 2 barky child pongs fumet
#> 3 banks porgy child fumet
#> 4 porks chant bilgy fumed
#> 5 parky bongs child fumet
#> 6 porky bangs child fumet

letter_freq
#>  [1] "s" "e" "a" "o" "r" "i" "l" "t" "n" "u" "d" "y" "c" "p" "m" "h" "g" "b" "k"
#> [20] "f" "w" "v" "z" "j" "x" "q"
head(orthogonal_words[[5]])
#>   word1 word2 word3 word4 word5
#> 1 chunk gymps waltz fjord vibex
#> 2 glent brick jumpy waqfs vozhd
#> 3 gucks waltz fjord vibex nymph
#> 4 blunk cimex grypt waqfs vozhd
#> 5 glent prick jumby waqfs vozhd
#> 6 bling treck jumpy waqfs vozhd
```

# Tweetable Wordle Game Engine

A playable game of Wordle in a tweet.

This was an exercise to see if I could simplify the WordleGame into just
280 characters. It’s mostly unreadable, and lacks safety checks, but it
works!

``` r
#RStats #wordle in a tweet
s=\(x)el(strsplit(x,''))
t=s(sample(grep("^[a-z]{5}$",readLines('/usr/share/dict/words'),v=T),1))
while(1){g=s(readline("? "))
M=t==g
r=which(!M)
for(i in r)for(j in r)if(g[i]==t[j]){M[i]=2;r=r[r!=j]}
cat(paste0('\033[48;5;',c(249,46,226)[M+1],'m ',g))}
```

<img src="man/figures/tweetable.png" />

# Expert Function: `filter_words()`

The `WordleHelper` R6 class is just a stateful wrapper around a core
function called `filter_words()`.

In general you wouldn’t need to call this function for solving a Wordle
puzzle but it might come in handy for other word puzzles.

In this example, I’m searching for a word:

-   with 9 letters
-   starting with `p`
-   containing `v` and `z` somewhere, but not as the first letter
-   containing only one `z`
-   without an `a` or an `o` in it

``` r
words <- readLines("/usr/share/dict/words")

filter_words(
  words            = words,
  exact            = "p........",
  wrong_spot       = c("vz", "", "", "", "", "", "", "", ""),
  min_count        = c(v = 1),
  known_count      = c(z = 1, a = 0, o = 0)
)
```

    #> [1] "pulverize"

## Acknowledgements

-   R Core for developing and maintaining the language.
-   CRAN maintainers, for patiently shepherding packages onto CRAN and
    maintaining the repository
