# Dice Examples

hypothetical examples of how dice would look

## Hello, World

`hello.dice`

```zig
const std = @import("std");

const main = syscall fn() void {
    std.debug.print("Hello, World", .{});
};
```

## Hello, World Lib

`hello-lib.dice`

```zig
const std = @import("std");

pub export const helloWorld = syscall fn() void {
    std.debug.print("Hello, World", .{});
};
```

This introduces `pub` and `export`. `pub` makes the function available to other
source files via `@import` and `export` notates that this function is part of
the public interface for the library artifact that this project creates. Because
dice follows the C ABI, `export`ed functions cannot have non-c-compatible
features in the function signature.

## Hello, World (no syscalls, print into buffer)

`hello-lib.dice`

```zig
const std = @import("std");

// TODO: "const" could be inferred here. make it not necessary?
pub export const helloWorld = fn(allocator: @allocator) void {
    std.fmt.allocPrint(allocator, "Hello, World", .{});
};
```

TODO: figure out how @allocator works. It's desirable to inject code with
different domain (for example `heap`) into otherwise non-heap code which is
generic over allocation. If it accepts some sort of `fn*(...)` then it won't
accept a `heap fn*()`. Functions pointers passed in need to be ownership checked
all the way down so we can determine that they are passed in by the user, and
then we can allow the functions to be of other domains. However then the
`helloWorld` function shown above would also need something like `inheritdomain`
which causes it to be the sum of all domains of any function pointers passed in.
