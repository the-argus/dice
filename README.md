# dice

A hypothetical programming language for making videogames. Intended to meet the
needs of both gameplay and engine programmers.

The thing that makes this language idea somewhat unique is the [domain keywords](./DOMAIN_KEYWORDS.md).

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

I want a language can be used seamlessly both when prototyping and
doing gameplay code, and when writing highly performant engine code.
C++ sort of achieves this by just having the features of a million different
languages, which makes it usable at different abstraction levels. However,
those abstraction levels often sort of fight each other RAII discourages
allocating and initializing separately, for example. Big C++ wants you to
`std::make_unique` but I can (and might want to) `malloc`.

I want Zig levels of compile time logic and type introspection.

I want to encourage and assist documentation as well as Rust does.

I want error messages as good as Rust's.

I want access to implicit control flow (unlike Zig) but only _sometimes_.

(Related) In Rust, I can turn the part of my brain that checks for potential segfaults,
as long as I'm outside of an `unsafe` block. In Zig, I can turn off the part
of my brain thinking about what operator overloads, destructors, getters, etc are
_actually_ being called on any given line of code, because Zig has no implicit
control flow. I want a language where I can declare "this file will not have any
of X" and I can turn off that part of my brain.

I want user-customization for these declarations, like maybe a game would have
files marked "noref" which is some custom keyword defined by some config file,
which prevents code in that file from storing references to certain game entity
types.

I want userland implementations of events/observer, oop, etc. to be in the
stdlib. There should be enough metaprogramming available that you can make your
code "clean" any which way you want.

I want custom metaprogramming and generics to generally be discouraged: it adds
overhead complexity which may be good for you but not a downstream developer
or future maintainer.

I want to treat language design as a game. Give developers encouragement to
program in the simplest way possible but still provide complex features. An example
of **not** this is Zig, which refuses to add complex type-constraining features such
as traits/interfaces because it encourages over-engineering and over-architecting
and because such type constraining code can be difficult to maintain and
understand. I think it also is an effort to keep things simple. I want to try to
_encourage_ simplicity but still make complexity available for when it is needed
in domain-specific cases.

I want access to borrow checking in areas where that sort of memory security
is necessary. I think that to have borrow checking work, it needs to be on by
default. As a result, I think there will simply be a lot of code marked with
some keyword like `noborrow`. Not exactly what I want but I think it's the best
solution. Basically: I want a language which is mostly a more usable version of
unsafe rust.

I want an option to do borrow checking with only explicit control flow. So
instead of implicitly destroying a mutable reference at the end of a scope, the
compiler could instead give you an error if you don't manually destroy it.

I want the ecosystem of libraries to have API complexity proportional to how many
layers of dependencies that library has. For example, a libcurl equivalent
shouldn't have an API which uses a bunch of implicit control flow and type
introspection to create userland OOP. It should just provide some functions,
which do syscalls to make internet things happen. Other packages can provide a
"nice" interface for the actual functionality it provides. If a package is a
CLI matrix chat utility library which depends on curses and libcurl, then it
is okay for that to create abstractions, because it will be used for a more
limited (and therefore better known) domain of problems.

So dice should small, simple, performant, and readable by default, but allow for
annotations which declare the domain of code and give access to larger or
different featuresets. These features should be encouraged to be used sparingly,
when working in a limited domain where the feature is known to always be helpful.

## Rules of Dice

Goals for the language.

**Tell the compiler everything.**

This means that you should have to write _extremely_ detailed information
about any variable behavior. You should be able to annotate what code paths are
hot and what are cold, for example. You should have to annotate exactly what
type or types are on the other end of a pointer. You should be able to annotate
exactly the set of possible functions that a function pointer can go to. You
should have to very particularly constrain generic code.

**Focus on tooling.**

Dice _must_ have a responsive, reliable, and accurate LSP implementation. The
language server should be able to reason about generic code, instead of simply
providing no information as is with clangd and a C++ template like
`template <typename T>`.

**Use the C ABI and existing tooling.**

GDB should work with dice.

**Do not provide virtual functions.**

This rule is more of an example which epitomizes some of the other rules. But it's
also a promise.

Virtual functions are a particular type of function pointer, meant to allow a
particular type of programming. This means that function pointers already provide
what virtual functions achieve. Adding specifically virtual functions increases
complexity of the language for users and for implementers of its spec.

**No preprocessor.**

Language customization should be easy to debug. We're not trying to have a repeat
of Qt or GObject. Arbitrary compile time code execution and introspection ala Zig.

**Be extensible for games, but be simple and readable for libraries.**

There is a distinct difference between an application (designed for users first,
then maintainers) and a library (designed for users first, then downstream
programmers, then maintainers). Maybe you should be able to declare that your
artifact is a library or application, and get different compiler warnings/analysis
depending on each?

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
