## Installation & Usage 

```
$ cd workshop
$ raco pkg install --auto
$ racket main.rkt
"Hyaaaaar"
"Guys, I'm not a pony."
```

### 1. `main.rkt`, `expander.rkt` and `ranch.rkt`.

By means of `(require (only-in "ranch.rkt" the-ranch))` `main.rkt` "imports" `ranch.rkt`:
```
#lang s-exp workshop/expander

(ranch
  (ponies
    (pony firestorm "Hyaaaaar")
    (pony rarity "I'm so shiiinyyy")
    (pony dash "Look at dat speed!")
    (pony dog "Guys, I'm not a pony.")))
```
which is expanded by means of [ `expander.rkt` ](https://github.com/mkohlhaas/racket-custom-language-example/blob/master/workshop/expander.rkt) to something like :

```
(provide the-ranch)

(define (the-ranch pony-name)
  (cond [(eq? pony-name 'firestorm') "Hyaaaaar"]
 	[(eq? pony-name 'rarity') "I'm so shiiinyyy"] 
	[(eq? pony-name 'firestorm') "Look at dat speed!"]
	[(eq? pony-name 'firestorm') "Guys, I'm not a pony."]
	[else 
	   "This pony does not exist!" ]))
```
`expander.rkt` reads the whole file `ranch.rkt` (apart from the #lang line) as `expr` into its `module-begin` macro and expands it to the aforementioned form by means of a `ranch` macro.
`the-ranch` is just a lambda/function that can be used in `main.rkt`, eg:

```
(the-ranch 'firestorm)
```
(Actually, `main.rkt` only imports `the-ranch` from `ranch.rkt`. Therefore I put imports above in quotes.)

### 2. `main.rkt`, `expander.rkt`, `fancy-ranch.rkt`, `pony.rkt`, `reader.rkt`, `parser.rkt`.

In this scenario we build our own language on top of the s-expr form of 1. with `fancy-ranch.rkt`, `pony.rkt`, `reader.rkt`, `parser.rkt`.
`expander.rkt` is the same as in 1. and `main.rkt` has to be changed in order to use the new language. (We'll see how in a moment.) `ranch.rkt` is not used any more.

We define our language in Extended-Backus-Naur form in `parser.rkt` which provides a `parse`-function to the lexer `reader.rkt`.
See `parse` call in function [ `pony-read-syntax` ]( https://github.com/mkohlhaas/fosdem-2019-talk/blob/master/workshop/reader.rkt#L17 ).
The reader is indirectly used in `fancy-ranch.rkt` (we'll see in a moment how)

```
#lang workshop/pony
R[
  P[ firestorm "Hyaaaaar"
  P[ rarity "I'm so shiiinyyy"
  P[ dash "Look at dat speed!"
  P[ dog "Guys, I'm not a pony."
]R
```
to lex and parse it into the s-expr form of 1.

From there the expander takes over and does its work as in 1. The reader must provide two methods
`read` and `read-syntax`. See [ Master recipe in Beautiful Racket ]( https://beautifulracket.com/appendix/master-recipe.html#the-reader ).
But must do so - and this is important - as functions of a submodule. Therefore we create a dummy module in `pony.rkt` which just provides these two methods
in a submodule by means of the [ `reprovide-lang` package ](https://docs.racket-lang.org/reprovide/).
This `pony` dummy module is used in `fancy-ranch.rkt` and not the `reader` module directly.

Now we can use our own custom language in `main.rkt` using `fancy-ranch.rkt`:

```
(require (only-in "fancy-ranch.rkt" the-ranch))
```
As a reminder, previously we used the now superfluous `ranch.rkt` file.

When you run the `main.rkt` again you won't see any difference:
```
$ racket main.rkt
"Hyaaaaar"
"Guys, I'm not a pony."
```
