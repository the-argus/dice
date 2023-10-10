# Domain Keywords

A "domain keyword" is a keyword which specifies the feature needs of a block of
code. For example, if you need raw pointer access in rust, you declare a block
`unsafe`.

## Keywords

A list of planned domains.

### Type decorators

These are decorators which are special: they appear elsewhere but have special
meaning when applied to types.

`implicit`: When decorating functions in a type with `implicit`, you overload
operators. Define functions such as `$+` to achieve this.

### Block/Function decorators

These are decorators which must be applied to a block in order to get domain
specific features, but also can be used on function signatures to force callers
to be in a block with the same decorator in order to perform the call.

`noborrow`: no borrow checking. `noborrow` functions can only be called from
`noborrow` blocks. For types instantiated in regular blocks (not `unsafe` or
`noborrow`) they must be moved into the `noborrow` block in order to be passed
to any `noborrow` functions.

`unsafe`: raw pointer access and arithmetic, the `@bitCast` builtins, no borrow
checking, and the ability to call other `unsafe` functions. Also considered
`noborrow`.

`recurse`: when used on a function, allows recursion. when used on a block,
allows calling recursive functions.

`implicit`: types instantiated in an `implicit` block have their `destructor`
method called when they leave scope. You can also call other `implicit` functions
(such as overloaded functions). A function which uses `implicit` blocks must
also be marked `implicit`. It is a warning to be in a scope which is both `unsafe`
and `implicit`.

`heap`: allows the use of global OS allocators such as malloc,
as well as calling other `heap` functions. Functions which contain a `heap`
block must be marked `heap`. It is a warning for a `heap` function to be called
from an `implicit` block or vice versa.

`syscall`: allows the use of system call builtins. A function with a `syscall`
block must be marked `syscall`. A `heap` block is also considered `syscall`. It
is a warning for a `syscall` block to be called from an `implicit` block or
vice versa.

### Function decorators

Domain keywords which are placed in function signatures and affect the contents
of the whole function.

`generic`: allows passing in a type to a function, or allowing the types of
arguments to the function to be functions which take `infer T`. Also see
[GENERICS.md](./GENERICS.md)

`introspect`: allows the use of the `@Typeinfo` builtin to run procedural code
over a type at compile time.

`nosideeffects`: disallows access to static memory (global variables) and use of
any `unsafe` blocks or functions. You may not call functions which are neither
`nosideeffects` or `pure`.

`immutableargs`: functions decorated with this cannot accept mutable references,
and cannot call `unsafe` functions or use `unsafe`.

`pure`: Combination of both `nosideeffects` and `immutableargs`.

`async`: Function is called as a coroutine.

`panics`: able to use the `@panic` builtin and call other `panics` functions.

`hack`: able to use the `@mixin` builtin to inject code into library functions
and access non-pub functions from a different dice project. Also able to call
other `hack` functions. It is an error for a `hack` function to also be `export`.

## Keyword behavior

The following keywords "bubble up," ie. they can only be called from contexts
which also use that keyword:

- `implicit`
- `heap`
- `syscall`
- `hack` (can't be exported, also)
- `panics`

The following keywords can be used and "wrapped," ie. used from contexts which
don't share that keyword:

- `unsafe`
- `noborrow` (requires borrow-checked types to be moved in order to be used)
- `recurse`
- `async`
- `generic`
- `introspect`
- `nosideeffects`
- `immutableargs`
- `pure`
