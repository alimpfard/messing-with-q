# Messing with and Learning Q

This repo hosts my experiments and attempts at learning Q, as a programmer that knows _some_ Haskell and has his own weird functional language.

The source of learning Q is [Q For Mortals 3](https://code.kx.com/q4m3).

Each section in that document will be a section in here, with added comparisons (Haskell and Citron currently), and possibly my thoughts about the section, if any.

Note for the people reading the markdown: Since github does not have Citron syntax highlighting, some supported languages with similar-enough outputs are used to get syntax highlighting :)

## 1.1 Starting q

```q
"c"$0x57656c6c20646f6e6521
```

Okay, that's a little weird, but `"c"$` apparently turns the hex argument to a string of characters...I guess.

## 1.2 Variables

```q
a:42
```

Sure, why not.

## 1.3 Whitespace

> Accomplished q programmers view whitespace around operators as training wheels on a bicycle.

Uhhh...a little stupid, really, the whitespace is there for _you_ the writer, and for _me_ the reader.

Don't fuck both of us over with being stupid like this.

## 1.4 The Q Console

> To obtain official console display of any q value, apply the built-in function show to it.
```q
show a:42
42
```

Alright, that seems nice, afterall, Haskell has the same thing:
```hs
show 42
42
```

## 1.5 Comments

Forward slash...sure, I guess, they could just use two of those and get a nice division operator, but who am I to decide.

> At least one whitespace [...] {must exist before the comment}

Okay, I guess that's how they get that `+/` thing working.

```q
a:42/ intended to be a comment
'
```

Now that error message is just...I shall let them explain themselves

> The q gods have no need for explanatory error messages or comments since their q code is perfect and self-documenting

No kidding.

## 1.6 Assignment

> A variable is not explicitly declared or typed

That's pretty normal for a dynamic language, sure.

> In q an assignment carries the value being assigned [...]
```q
1+a:42
43
```

Not particularly unusual, a comparison with Citron:

```kt
1 + var a is 42
43
```

## 1.7 Order of Evaluation

This is pretty hairy, but also pretty uniform.
Everything is right-associative, pretty cool

compared to Haskell with comparatively complex rules about associativity, and Citron with _extremely_ complex rules regarding associativity, it's a nice little thing.

## 1.8 Data Types 101

Okay, integer literals are 64-bit, good to know.
still doesn't explain how they got that `0x57656c6c20646f6e6521` (80 bits) to work, but let's roll with it for now.

Compared to Haskell where all numeric literals are of a polymorphic type, this is somewhat of a letdown, but ok.

It also has native types and literals for date and timespan, pretty cool.

Also, symbols. yay.
```q
`aapl / You can tell the roots of this language here

`aapl = `apl
0b
```

Haskell has no particular way of doing symbols as q does, but `'Thing` is still somewhat relevant.

They seem to share the same semantics as Symbols in Citron:
```
\thing

\thing = \other-thing
False
```

## 1.9 List 101

lispy lispy lists...with extra semicolons?

```q
(1; 1.2; `one)
1
1.2
`one
```

They're not cons-cells, and not linked-lists...okay then.

Apparently if the types are homogeneous, the semantics differ:
```q
(1; 2; 3)
1 2 3
```

...and they also differ between differnt types:
```q
(1b; 0b; 1b)
101b
```

Okay then weirdo.



Ah. finally, some functions
```q
til 10
0 1 2 3 4 5 6 7 8 9
```

Neat, a range/generator function...I think.

Similar to `[0..9]` in Haskell or `0..10` in Citron, I guess. nothing interesting here.

```q
1 + til 10
1 2 3 4 5 6 7 8 9 10
```

And now it shows its APL influence, pretty cool.
you can't do _that_ in Haskell (without extra definitions anyway)

Citron has a similar thing as `Data/Vectorized`, though for a different purpose (SIMD):
```kt
Vector[x,, 0..9] + 1
Vector[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

Adding lists:
```q
1 2 3, 4 5
1 2 3 4 5
```

Eh, looks ok I guess

In Haskell, we can use `(++) :: [a] -> [a] -> [a]`
```hs
[1, 2, 3] ++ [4, 5]
[1, 2, 3, 4, 5]
```

and Citron just overloads `+`:
```kt
[1, 2, 3] + [4, 5]
[1, 2, 3, 4, 5]
```


Item extraction:

```q
2 # til 10
0 1

-2 # til 10
8 9
```
Now this is interesting, a slice op from both ends. Haskell doesn't really have this as lazy lists are bad things to index from the end, but you can still do it:
```hs
lasts n xs = drop (length xs - n) xs
```

Citron doesn't do these, but `xs viewFrom: xs count - n` could do the trick.

A more interesting facet of this `#` operator is that it actually cycles if you run out of items. pretty cool.

You can also select from the list by applying it to a list of indices, neat.

## 1.10 Functions 101

Only buitlin functions can be applied as infix. sadface.

As compared to Haskell that allows you to configure even the fixity and precedence of functions, this is definitely pretty mediocre.

They come in two flavours, with implicit mathematical variables (!) and explicit parameter lists:
```q
{x*y}
{[a;x] a*x}
```

The implicit version is _ordered_, that is, the arguments are named in order `x`, `y`, and `z`...Okay.

Here's the haskell version, for comparison:
```hs
\a x -> a * x
```

And the citron version:
```kt
\:a:x a * x
```

Application can be done with square brackets (very similar to citron here):
```q
{x*y}[1;4]
4
```

In citron:
```kt
{\:x:y x * y}[1, 4]
4
```

of course, you can use the operators directly as functions too, these are functional languages after all:

Q:
```q
(*)[1;4]
4
```
Haskell:
```hs
(*) 1 4
4
```
And Citron requires the hosting object name, but is otherwise similar:
```kt
(Number:::'*')[1, 4]
4
```

## 1.11 Functions on Lists 101

Blah blah, builtins apply to atoms, not interesting anymore.

what _is_ interesting is this:
```q
0 +/ 1 2 3 4 5
15
```

Now what's going on there? apparently appending a `/` to a function makes an iterator version of it, i.e. something that folds. neat.

```q
(*/) 1 + til 10
3628800
```

This can be expressed in Haskell with `foldl1`, but the (1 + ...) part remains a list comprehension, or just a map:
```hs
foldl1 (*) $ fmap (+1) [0..9]
3628800
```

In citron, this looks like
```kt
Vector[x,, 0..9] + 1 foldl: (Number:::'*')
3628800
```

#### To Be Expanded...
