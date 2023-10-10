# Generic Programming in Dice

In order to make code generic over types, you first need to declare a function
`generic` and then either pass in a type with `comptime type_name: type` or
`variable_name: infer type_name`.

Here is an example of a generic method which adds any two things which have the
plus operator:

```zig
// needs introspect for access to @hasDecl and generic for access to accepting
// types as arguments.
const additionTrait = introspect generic fn(comptime T: type) bool {
    return @hasDecl(T, "$+");
};

// needs to be implicit for operator overloading and generic for receiving types
// note that the `additionTrait` function must take only compile time known
// parameters in order to be used in this context
pub const addThings = implicit generic fn(thing1: infer additionTrait(T), thing2: T) T {
    // it is a compile error to use anything other than the + operator on these
    implicit {
        return thing1 + thing2;
    }
};
```
