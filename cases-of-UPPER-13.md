# Definitely How What Where, Who?

> This is part thirteen in the ["Cases of UPPER"](https://dev.to/lizmat/series/35190) series of blog posts, describing the Raku syntax elements that are completely in UPPERCASE.

This part will discuss the various introspection methods that you can use on objects in the [Raku Programming Language](https://raku.org).  Note that in some documentation these methods are also referred to as [Metamethods](https://docs.raku.org/language/mop#Metamethods).

## Turtles All The Way Down

In Raku everything is an object, or can be thought of as an object.  An object is an instantiation of a class (usually made by calling the `new` method on it).  A class is represented by a so-called ["type object"](https://docs.raku.org/language/objects#Type_objects).  Such a type object in turn is an instantation of a so-called [meta class](https://docs.raku.org/language/mop).  And these meta classes are themselves built out of more primitive representations.

> Going this deep would most definitely be out of scope for these blog posts.  But yours truly does intend to go there at some point in the future.

## WHAT

The [`WHAT`](https://docs.raku.org/syntax/WHAT) method returns the [type object](https://docs.raku.org/syntax/type%20object) of the given invocant.  Not much else to tell about it really.
```raku
say 42.WHAT;     # (Int)
say "foo".WHAT;  # (Str)
say now.WHAT;    # (Instant)
```

## HOW

The [`HOW`](https://docs.raku.org/syntax/HOW) method returns the meta-object of the class of the given invocant.  The `HOW` (b)acronym stands for "**H**igher **O**rder **W**orkings".  It allows one to introspect the class of the invocant.
```raku
say 42.HOW.name(42);        # Int
say "foo".HOW.name("foo");  # Str
say now.HOW.name(now);      # Instant
```
Note that the invocant of the `HOW` method needs to be repeated in the introspection method's call as the first argument.  Why?  Well, this is really to be *possibly* compatible with *future* versions of Raku.

Since one is usually only interested in the introspection aspect of `HOW`, a shortcut method invocation was created that allows one to directly call the introspection method **without** needing to repeat oneself: [`.^`](https://docs.raku.org/language/operators#methodop_%2E^):
```raku
say 42.^name;     # Int
say "foo".^name;  # Str
say now.^name;    # Instant
```
Some other common introspection methods are [`mro`](https://docs.raku.org/type/Metamodel/C3MRO#method_mro) (showing the base classes of the class of the value) and [`methods`](https://docs.raku.org/routine/methods) (which returns the `method` objects of the methods that can be called on the value):
```raku
say 42.^mro;           # ((Int) (Cool) (Any) (Mu))
say 42.^methods.sort;  # (ACCEPTS Bool Bridge Capture Complex...
```
Note that these meta-classes are classes themselves, so can have meta-methods on them called as well:
```raku
say 42.HOW.^name;  # Perl6::Metamodel::ClassHOW
```
> Yeah, there's still some legacy code that will need renaming under the hood!

## WHERE

The [`WHERE`](https://docs.raku.org/routine/WHERE) method returns the memory address of the invocant.  It is of limited use in the Rakudo implementation as the memory location of an object is **not** guaranteed to be constant.  As such, it is intended for (core) debugging only.
```raku
say 42.WHERE;  # 2912024602280 (or some other number)
```

## VAR

In the [previous blog post](https://dev.to/lizmat/store-proxy-fetch-a07) the `Scalar` object was described.  But the `Scalar` objects are nearly invisible.  How can one obtain a `Scalar` object from a given variable?  And find out its name from that?

The "secret" to that is the [`VAR`](https://docs.raku.org/syntax/VAR) method.
```raku
my $a;
say $a.VAR.^name;  # Scalar
```
Each `Scalar` object provides at least these [introspection methods](https://docs.raku.org/type/Scalar#Introspection): [`of`](https://docs.raku.org/type/Scalar#method_of), [`name`](https://docs.raku.org/type/Scalar#method_name), [`default`](https://docs.raku.org/type/Scalar#method_default) and [`dynamic`](https://docs.raku.org/type/Scalar#method_dynamic).
```raku
my Int $a is default(42) = 666;
say $a;              # 666
say $a.VAR.of;       # (Int)
say $a.VAR.name;     # $a
say $a.VAR.default;  # 42
say $a.VAR.dynamic;  # False
```
The `of` method returns the constraint that needs to be fulfilled in order to be able to assign to the variable.  The `name` method returns the name of the variable.  The `default` method returns the default value (`Any` if none is specified).

The `dynamic` method returns `True` or `False` whether the variable is visible for dynamic variable lookups.  This usually only returns `True` for variables with the [`*` twigil](https://docs.raku.org/language/variables#The_*_twigil).

## WHO

The [`WHO`](https://docs.raku.org/syntax/WHO) method (for "who lives here?) is actually a bit of a misnomer.  It should probably have been called `OUR` because it returns the [`Stash`](https://docs.raku.org/type/Stash) of the type object of the invocant.  And a stash is an object that does the `Associative` role, and as such can be accessed as if it were a `Hash`.  And the stash of a type object is the same namespace as `our` inside that package.

So for instance if you would like to know all classes that live in the `IO` package:
```raku
say IO.WHO.keys.sort;  # (ArgFiles CatHandle Handle Notification Path Pipe Socket Spec Special)
```
Of course, you would know them more by their complete names such as `IO::ArgFiles`, `IO::CatHandle`, `IO::Handle`, etc.  In fact the [`::`](https://docs.raku.org/language/packages#index-entry-::) delimiter is shortcut for using `WHO`:
```raku
say IO.WHO<Handle>;  # (Handle)
say IO::Handle;      # (Handle)
```
And that goes even further: `foo::` is just short for `foo.WHO`:
```raku
say IO::.keys.sort;  # (ArgFiles CatHandle Handle Notification Path Pipe Socket Spec Special)
```
A little closer to home: how would that look in a package that you define yourself and have an [`our`](https://docs.raku.org/syntax/our) scoped variable in there:
```raku
package A {
    our $foo = 42;
}
say A.WHO<$foo>;  # 42
say A::<$foo>;    # 42
say $A::foo;      # 42
```
> The careful reader will have noticed that [`package`](https://docs.raku.org/language/packages) was used in the example.  A very simple reason: `class`, `role`, `grammar` are all just packages with different `HOW`s.  And `WHO` doesn't care why kind of `package` it is.

## REPR

The [`REPR`](https://docs.raku.org/language/traits#is_repr_and_native_representations%2E) method returns the *name* of the memory representation of the class of the invocant.  For most of the objects this is "P6opaque".

> This is basically the representation used by `class` and its attribute specifications.

The [`NativeCall`](https://docs.raku.org/language/nativecall) module provides a number or alternate memory representations, such as [`CStruct`](https://docs.raku.org/language/nativecall#Structs), [`CPointer`](https://docs.raku.org/language/nativecall#Basic_use_of_pointers) and [`Cunion`](https://docs.raku.org/language/nativecall#CUnions).  Native arrays also have a different representation (`VMArray`).
```raku
say 42.REPR;  # P6opaque
my int @a;
say @a.REPR;  # VMArray
```
When a class is defined, it gets the `P6opaque` representation by default.
```
class Foo { }
say Foo.REPR;  # P6opaque
```
> Unless one is doing very deep core-ish work, how an object is represented in memory should not be of concern to you.

## DEFINITE

The [`DEFINITE`](https://docs.raku.org/syntax/DEFINITE) method returns either `True` or `False` depending on whether the invocant has a concrete representation.  This is *almost always* the same as calling the [`defined`](https://docs.raku.org/routine/defined) method.  But in some cases it makes more sense in Raku to return the opposite with the `defined` method.

An example of this is the `Failure` class:
```raku
say Failure.new.DEFINITE;  # True
say Failure.new.defined;   # False
```
In general the `defined` method should be used.  The `DEFINITE` method is intended to be used in very low-level (core) code.  It's not all uppercase for nothing!

> The reason `Failure.new.defined` always returns `False` is to make it compatible with [`with`](https://docs.raku.org/syntax/with%20orwith%20without).

## Macroish

All of the introspection "metamethods" described in this blog post are actually parsed as macros directly generating low-level execution opcodes.  This is really necessary in some cases (as otherwise information can be lost), and in other cases it's just for performance.

This doesn't mean that it's not possible to call this introspection functionality as a method: you can.  For example:
```raku
my $a;
say "macro: "  ~ $a.VAR.name;      # macro: $a
say "method: " ~ $a."VAR"().name;  # method: $a
```
Furthermore for consistency, the same functionality of these methods is **also** available as subroutines.
```
my $b;
say "sub: " ~ VAR($b).name;  # sub: $b
```
So if you're more at home in imperative programming, you can do that as well!

## Conclusion

This concludes the thirteenth episode of cases of UPPER language elements in the Raku Programming Language, the sixth episode discussing interface methods.

In this episode the following macro-like introspection methods were discussed (in alphabetical order): `DEFINITE`, `HOW`, `REPR`, `VAR`, `WHAT`, `WHERE`, `WHO`.

Stay tuned for the next episode!
