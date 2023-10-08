# dice

A hypothetical programming language for making videogames. Intended to meet the
needs of both gameplay and engine programmers.

## Goals of this repo

- Write down my thoughts on programming languages in general
- Write down my thoughts on the needs of game developers vs. other software developers
- Write down my thoughts on how my ideal language would be maintained
- Provoke thought and dicussion (I want to hear what other game devs think about
  these concerns)
- Maybe one day actually make a compiler for this language
  - if it gets to the point where I feel like people other than just me want/need
    it

## What I Want

I would like a language which by default has similar capacities to Zig, but with
the possibility of adding implicit control flow. The idea being that the language
is low-level by default, but it's possible to provide an API which is intended to
be (and annotated to be) high-level.

This is so that the language can be used seamlessly both when prototyping and
doing gameplay code, and when writing highly performant engine code.

Imagine Rust's `unsafe` keyword, but instead it's something like `implicit`.
maybe a `cold` keyword to say "I can dereference this function pointer which has
no constraints on its destination" or a `loadtime` keyword to say "this doesn't have
to complete in 0.016ms", or a `frametime` keyword to say the opposite.

I want to be able to say "this function is for prototyping game behavior, I want
access to these language features" and also be able to say "this is a function
which is intended to be a public library API, so restrict the use of these
newer features."

Maybe there could be a `domains.dice` file in your project, where you can declare
keywords like `loadtime` and then declare the language feature constraints that
should be put on functions annotated with that keyword. And then also have some
of these keywords in the stdlib, like `unsafe` and `implicit`.

So dice should small, simple, performant, and readable by default, but allow for
annotations which declare the domain of code and give access to larger or
different featuresets.

I also want access to borrow checking in areas where that sort of memory security
is necessary. Given Zig levels of compile time procedural logic and also implicit
control flow, it should be possible to implement a borrow checker in userland.

## Rules of Dice

Goals for the language.

**Tell the compiler everything.**

This means that you should be able to write _extremely_ detailed information
about any variable behavior. You should be able to annotate what code paths are
hot and what are cold, for example. You should be able to annotate exactly what
type or types are on the other end of a pointer. You should be able to annotate
exactly the set of possible functions that a function pointer can go to. You
should be able to very particularly constrain generic code.

**Focus on tooling.**

Dice _must_ have a responsive, reliable, and accurate LSP implementation. The
language server should be able to reason about generic code, instead of simply
providing no information as is with clangd and a C++ template like
`template <typename T>`. Further, the language server should be able to compare
how a type is used vs. how it is constrained in order to assist the process of
writing and maintaining type-constraining code.

**Do not provide virtual functions.**

This rule is more of an example which epitomizes some of the other rules. But it's
also a promise.

Virtual functions are a particular type of function pointer, meant to allow a
particular type of programming. This means that function pointers already provide
what virtual functions achieve. Adding specifically virtual functions increases
complexity of the language for users and for implementers of its spec.

**Provide meta-programming facilities complex enough that virtual functions can
be implemented in userland.**

Users should be able to customize the language to their needs.

**Compiler error messages as helpful as Rust's.**

Sometimes a language is just the gold standard in a particular area.

**No preprocessor.**

Language customization should be easy to debug. We're not trying to have a repeat
of Qt or GObject. Arbitrary compile time code execution and introspection ala Zig.

**Metaprogramming should primarily encouraged for internal, or domain-specific use.**

It should be discouraged for a library to expose metaprogrammed types or
functions to its public API, because then it becomes less readable for users.
Metaprogramming is a way to specialize the language for your needs. A game can
do this left and right: do what's best for the designers. No one depends on your
game. But a person maintaining a physics engine library should be held to a different
standard. If they want to add OOP and vtables to their project, fine. But if their
API requires you to take on their complex metaprogrammed types into your project,
that should incur a warning, at least.

Possible solution: Provide rich information about how custom/metaprogrammed a
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

Maybe force a `generic` declaration in the style of Rust's `unsafe`.

**Be extensible for games, but be simple and readable for libraries.**

There is a distinct difference between an application (designed for users first,
then maintainers) and a library (designed for users first, then downstream
programmers, then maintainers). The previous point may already address this, but
it's worth pointing out. Maybe you should be able to declare that your artifact is
a library or application, and get different compiler warnings/analysis depending
on each?

**Rust-style, first-class documentation generation, with an exception.**

Do it the way rust does it. But also allow for annotating a file as "final" or
"user" or something, to say "this is not intended to be used by anyone else",
allowing bypass of compiler warnings about a lack of complete docstrings.

**Concern for backwards compatibility is proportional to the complexity of the
language feature.**

Ideally, there would be a very small subset of the language, with a similar featureset
to C, which is guaranteed to never change. Libraries made with this subset would
always compile with all future compiler versions. Also, libraries made with this
subset would have a language complexity rating of 0. Other features may become
solidified, but only after years of real-world use.

The reason for this is so that library developers can have their compatibility cake,
and game developers can just pin a version of the compiler and customize using
advanced features to their heart's content. Of course, library developers then
can't use advanced features, but dice prioritizes the experience of the game
developer.

**Encourage the simplest approach to problems.**

Provide compiler warnings and information if you have a generic type which is
instantiated with only one type, or a small generic type only instantiated twice.
Provide warnings for complex code (also see clang-tidy's cognitive complexity
analysis).

**Use the C ABI and existing tooling.**

GDB should work with dice.
