# Positional Methods

> This is part nine in the ["Cases of UPPER"](https://dev.to/lizmat/series/35190) series of blog posts, describing the Raku syntax elements that are completely in UPPERCASE.

This part will discuss the interface methods that one can implement to provide a custom [`Positional`](https://docs.raku.org/type/Positional) interface in the [Raku Programming Language](https://raku.org): [postcircumfix [ ]](https://docs.raku.org/language/operators#postcircumfix_[_]), the universal interface for positional access (aka the "array indexing operator").

## Some background

The `Positional` role is really just a marker.  It does **not** enforce any methods to be provided by the consuming class.  So why use it?  Because it is the constraint that is being checked for any variable with a `@` [sigil](https://docs.raku.org/language/glossary#Sigil).  An example:
```raku
class Foo { }
my @a := Foo;
```
will show:
```
Type check failed in binding; expected Positional but got Foo (Foo)
```
However, if we make that class ingest the `Positional` marker role, it works:
```raku
class Foo does Positional { }
my @a := Foo;
say @a[0];  # (Foo)
```
and it is even possible to call the postcircumfix `[ ]` operator on it!

So why is that possible?  That's really possible because each item in Raku can be considered as a single element list.  And thus the first element (aka `[0]`) will **always** provide the invocant.
```raku
say 42[0];  # 42
say 42[1];  # Index out of range. Is: 1, should be in 0..0
```
Which in turn is why the `Positional` role does not enforce any methods.  Because they are always already provided by the [`Any` class](https://docs.raku.org/type/Any), so it would *never* complain about methods *not* having been provided.

## Postcircumfix [ ]

The `postcircumfix [ ]` operator performs all of the work of slicing and dicing objects that perform the `Positional`, and handling all of the adverbs: `:exists`, `:delete`, `:p`, `:kv`, `:k`, and `:v`.  But it is completely agnostic about how this is actually done, because all it does is calling interface methods that are (implicitely) provided by the object.  For instance:
```raku
say 42[0];  # 42
```
is actually doing:
```raku
say 42.AT-POS(0);
```
under the hood.  So these interface methods are the ones that actually know how to work on an [`Array`](https://docs.raku.org/type/Array), a [`List`](https://docs.raku.org/type/List) or a [`Blob`](https://docs.raku.org/type/Blob).

## The Interface Methods

Let's introduce the cast of this show:

- [`AT-POS`](https://docs.raku.org/language/subscripts#method_AT-POS)
- [`EXISTS-POS`](https://docs.raku.org/language/subscripts#method_EXISTS-POS)
- [`DELETE-POS`](https://docs.raku.org/language/subscripts#method_DELETE-POS)
- [`ASSIGN-POS`](https://docs.raku.org/language/subscripts#method_ASSIGN-POS)
- [`BIND-POS`](https://docs.raku.org/language/subscripts#method_BIND-POS)
- [`STORE`](https://docs.raku.org/language/subscripts#method_STORE)
- [`elems`](https://docs.raku.org/routine/elems#(Subscripts)_method_elems)

### AT-POS

The `AT-POS` method is the most important method of the interface methods: it is expected to take the integer index of the element to be returned, and return that.  It should return a container if that is appropriate, which is usually the case.  Which means you probably should specify [`is raw`](https://docs.raku.org/routine/is%20raw) on the `AT-POS` method if you're implementing that yourself.

### EXISTS-POS

The `EXISTS-POS` method is expected to take the integer index of an element, and return a [`Bool`](https://docs.raku.org/type/Bool) value indicating whether that element is considered to be existing or not.  This is what is being called when the [`:exists`](https://docs.raku.org/language/subscripts#:exists) adverb is specified.

### DELETE-POS

The `DELETE-POS` method is supposed to act very much like the `AT-POS` method.  But is also expected to remove the element so that the `EXISTS-POS` method will return `False` for that element in the future.  This is what is being called when the [`:delete`](https://docs.raku.org/language/subscripts#:delete) adverb is specified.

### ASSIGN-POS

The `ASSIGN-POS` method is a convenience method that *may* be called when assigning (`=`) a value to an element.  It takes 2 arguments: the index and the value.  It functionally defaults to `object.AT-POS(index) = value`.  A typical reason for implementing this method is performance.

### BIND-POS

The `BIND-POS` method is a method that will be called when binding (`:=`) a value to an element.  It takes 2 arguments: the index and the value.  If not implemented, binding will always fail with an execution error.

### STORE

The `STORE` method is an optional method that must implemented if the `my @a is Foo = 1,2,3` and `@a = 1,2,3` syntax is to be supported by the class.  Should accept the values to be (re-)initializing with.

The `:INITIALIZE` named argument will be passed with a `True` value if this is the first time the values are to be set.  This is important if your data structure is supposed to be immutable: if that argument is `False` or not specified, it means a re-initialization is being attempted.

### elems

Although not an uppercase named method, it is an important interface method: it is expected to return the number of elements in the object.  Basically the highest index value + 1 that can be specified expecting to be returned a value.

## Handling simple customizations

Wow, that's a lot to take in!  But if it is just a simple customization you wish to do to the basic functionality of e.g. an array, you can simply inherit from the `Array` class.  Here's a simple, contrived example that will return any content of the array doubled as string:
```raku
class Array::Twice is Array {
    method AT-POS(Int:D $index) { callsame() x 2 }
}
my @a is Array::Twice = <a b c>;
say "\@a[$_] = @a[$_]" for ^@a;
```
will show:
```
@a[0] = aa
@a[1] = bb
@a[2] = cc
```
Note that in this case the [`callsame`](https://docs.raku.org/syntax/callsame) function is called to get the actual value from the array.

Another contrived example where the customization initializes the array with given values in random order (using [`.pick`](https://docs.raku.org/type/List#routine_pick):
```raku
class Array::Confused is Array {
    method STORE(\values) { self.Array::STORE(values.pick(*)) }
}
my @a is Array::Confused = <a b c>;
say "\@a[$_] = @a[$_]" for ^@a;
```
will show something like:
```
@a[0] = b
@a[1] = a
@a[2] = c
```
Note that this example uses the fully qualified method syntax (`self.Array::STORE`) to obtain the values to initialize with.

## Helpful modules

If you want to be able to do more than just simple modifications, but for instance have an existing data structure on which you want to provide a `Positional` interface, it becomes a bit more complicated and potentially cumbersome.

Fortunately there are a number of modules in the ecosystem that will help you creating a consistent `Positional` interface for your class.

### Array::Agnostic

The [`Array::Agnostic`](https://raku.land/zef:lizmat/Array::Agnostic) distribution provides a role with all of the necessary logic for making your object act as an `Array`.  The only thing you *must* provide, are the `AT-POS` and `elems` methods.  Your class *may* provide more methods for functionality or performance reasons.

### List::Agnostic

The [`List::Agnostic`](https://raku.land/zef:lizmat/List::Agnostic) distribution provides a role with all of the necessary logic for making your object act as a `List`.  The only thing you *must* provide, are the `AT-POS` and `elems` methods.  As with `Array::Agnostic`, Your class *may* provide more methods for functionality or performance reasons.

### Example class

A very contrived example in which a [`Date`](https://docs.raku.org/type/Date) object is going to represented by a 3-element array.  The class looks like:
```raku
use List::Agnostic;

class Date::Indexed does List::Agnostic {
    has $!date;
    method AT-POS(Int:D $index) {
        try (*.year, *.month, *.day)[$index]($!date)
    }
    method elems() { 3 }
    method STORE(\values) { $!date := Date.new(|values) // Date.today; self }
}
```
and can be used like this:
```raku
my @a is Date::Indexed = "2026-02-11";
say "\@a[$_] = @a[$_]" for ^@a;
```
which would show:
```
@a[0] = 2026
@a[1] = 2
@a[2] = 10
```
Note that the `STORE` method had to be provided by the class to allow for the `is Date::Indexed` syntax to work.

> If you're wondering what's happening with `try (*.year, *.month, *.day)[$index]($!date)`: that's a list built of 3 [`WhateverCode`s](https://docs.raku.org/type/WhateverCode) with the specified index selecting the appropriate entry and executing that with the `$!date` attribute as the invocant.  If that would fail (probably because the index was out or range), then the [`try`](https://docs.raku.org/syntax/try%20%28statement%20prefix%29) will make sure that that is caught and a `Nil` is returned.

## Conclusion

This concludes the ninth episode of cases of UPPER language elements in the Raku Programming Language, the second discussion interface methods.

In this episode the `AT-POS` family of methods were described, as well as some simple customizations and  some handy Raku modules that can help you create a fully functional interface: `List::Agnostic` and `Array::Agnostic`.

Stay tuned for the next episode!
