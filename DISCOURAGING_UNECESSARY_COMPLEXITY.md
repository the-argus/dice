# Discouraging Unecessary Complexity in Dice

Dice is intended to provide complex metaprogramming facilities. It could be very
easy as an intermediate programmer to be amazed by this and get caught up in
architecting a huge system and "making your own language" for your project. But
this is rarely necessary and creates a lot of technical debt.

Initially, I had the thought "Why worry about bad programmers? Just let the rest
of us figure out how to do it right?"
But now I think it's worthwhile. The design of a language has a lot to do with
the ergonomics, I think. If an advanced programmer is tired, then they're an
intermediate programmer. They might start making decisions without doing good
value judgements before accruing technical debt. They might not bounds-check
a span before iterating. But with the proper tools, we can make doing these
things thoughtless.

## Force libraries to be simple

A possible approach to discouraging unecessary complexity could be to just give
libraries some encouragement to not specialize too much. This won't have much effect
on application developers, although it depends on the solution.

### Possible Methods

1. Numbers Game
  People like numbers, right? Give them a printout/analysis of the complexity of
  their library. How "lightweight" it is. Let them slap it as a badge in their
  github readme.
  Provide rich information about how custom/metaprogrammed a
  function or type is. Give these primitives complexity scores. A function with
  only compile-time known arguments, which is instantiated once, and uses only
  primitive types is the simplest possible function. Then a function which is
  runtime. Then a function which has some compile time and some runtime arguments.
  Then a function which takes in a type (comptime known, of course). Then a
  function which takes in a _constrained_ type. Then a function which takes in a
  type which in turn contains a function pointer which this function calls. And so
  on. Then, provide some information whenever you compile an artifact about its
  "minimum complexity": the numeric score of the most complex type/function
  imported in that artifact. Then, when compiling a project and/or
  fetching/searching dependencies, you get a printout of information about the
  amount of complexity your project is incurring.
2. Domain keyword bubble-up
  If you have to declare a function `implicit` or `generic` in order to use generic
  types or special implicit control flow features and in order to call other
  `generic` or `implicit` functions, then maybe that would encourage people to
  simply be more conscious of the language features they're using?
