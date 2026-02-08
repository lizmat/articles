# Tweak Build

> This is part eight in the ["Cases of UPPER"](https://dev.to/lizmat/series/35190) series of blog posts, describing the Raku syntax elements that are completely in UPPERCASE.

This part will discuss the interface methods that one can provide to tweak the creation of objects in the [Raku Programming Language](https://raku.org).

## But first some background

If you are unfamiliar as to how Raku allows one to create classes, describe attributes and methods, it is recommended to first have a look at [Classes and objects - A tutorial about creating and using classes in Raku](https://docs.raku.org/language/classtut).

In short: by default objects are created with the [`.new`](https://docs.raku.org/syntax/new%20%28method%29#(Object_orientation)_new_(method)_new_(method)) (which is provided by the [`Mu` base class](https://docs.raku.org/type/Mu)).  This `.new` method takes named arguments and tries to match these with attribute names that are supposed to be set with `.new`.  Any unmatched named arguments are silently ignored. 

Attributes that are settable with `.new` that are **not** specified will have any default value applied to them.  A small example:
```raku
class Foo {
    has $.bar = 42;  # attribute + accessor
}
say Foo.new.bar;             # 42
say Foo.new(:bar(666)).bar;  # 666
```
Well, actually that is a little white lie.  But it *is* true in most cases.

The thing is that you can also specify your own `.new` method in your class, and **still** have all of the attribute initialization work done for you.  The reason for this is that it's not really the `.new` method that is doing the work, but the [`.bless`](https://docs.raku.org/routine/bless) that is also provided by the `Mu` class.

Functionally the `.new` method supplied by `Mu` is:
```raku
method new() { self.bless(|%_) }
```
although in reality it's a little bit complex more allowing for some optimizations.  As object creation occurs a **lot** when you're executing a Raku program, so it makes sense to try to optimize that!

> Note that methods in Raku always have a `*%_` (aka a ["slurpy hash"](https://docs.raku.org/language/signatures#index-entry-slurpy_argument)) added to their signature, unless there is already a slurpy hash in the signature.

An example of a custom `.new` method that takes a single positional argument:
```raku
class Foo {
    has $.bar = 42;
    multi method new($bar) {
        self.bless(:$bar, |%_)
    }
}
say Foo.new.bar;             # 42
say Foo.new(:bar(666)).bar;  # 666
say Foo.new(137).bar;        # 137
```
The reason that this *also* allows the named argument way, is because of the [`multi`](https://docs.raku.org/syntax/multi).  This **adds** a candidate to the existing dispatch table for the `.new` method, so the `Mu.new` candidate can still be dispatched to.  Without the `multi` there would only be the one `.new` method in the dispatch table for `class Foo`.

Also note the `|%_` in the `self.bless` call: this makes sure that any additional named argments are also passed to `.bless` if you specify a positional argument.

## BUILD vs TWEAK

Any class in Raku can have a [`BUILD`](https://docs.raku.org/syntax/BUILD) and/or a [`TWEAK`](https://docs.raku.org/syntax/TWEAK) method specified.  Both are called by the object building logic, but only if it is possible to call them.  Both methods receive the same arguments as having been (indirectly) passed to `.bless`.  The return value of these methods will be ignored.

The `BUILD` method is called **instead** of attribute initializations from the named arguments.  Specifying the `BUILD` method means needing to mimic **all** named argument setting in that `BUILD` method..

Any default values for attributes are assigned if the attribute did not receive a value yet (either from attribute initializations from the named arguments, or having been initialized in the `BUILD` method).

The `TWEAK` method is called as the **last stage** of object instantiation.  Nowadays it is the recommended way of altering the named argument -> attribute initialization logic for a class (after all initializations and default setting has been done).

## BUILDPLAN

The [Rakudo](https:://rakud.org) distribution comes with a handy introspection module that shows what steps are taken (in pseudo-code) in the creation of an instance of a class: `BUILDPLAN`.  It's use is pretty simple: `use BUILDPLAN class`.
```raku
class Foo {
    has $.bar = 42;
}
use BUILDPLAN Foo;
```
will show:
```
class Foo BUILDPLAN:
 0 nqp::getattr(obj,Foo,'$!bar') = :$bar if possible
 1 nqp::getattr(obj,Foo,'$!bar') = 42 if not set
```
Note that there are two steps:
- assign value of named argument "bar" to the attribute "$!bar" if specified
- assign value `42` to the "$!bar" attribute if it has not been set yet

Now, if we add a `BUILD` method to it:
```raku
class Foo {
    has $.bar = 42;
}
submethod BUILD() { }
use BUILDPLAN Foo;
```
the build plan changes to:
```
class Foo BUILDPLAN:
 0 call obj.Foo::BUILD
 1 nqp::getattr(obj,Foo,'$!bar') = 42 if not set
```
Note that the first step changed from "nqp::getattr(obj,Foo,'$!bar') = :$bar if possible" to "call obj.Foo::BUILD".  So instead of looking for a named argument "bar", it is now just calling the "BUILD" method.

Also note that second step stayed the same: so if the `BUILD` method doesn't assign anything to the `$!bar` attribute, the default value will *still* be set.

## Tricks and Tips

Historically the `BUILD` method was one of the first things actually implemented in Raku, and thus a lot of (older) code in the wild uses the `BUILD` method.  Since then many features have been added to the way one can specify attributes, obsoleting some of the common `BUILD` uses.

Here are some examples of outdated idioms:

### Making a named argument required

The `BUILD` can make a named argument mandatory by making it a required named argument by adding a `!` in the signature of `BUILD`:
```raku
class Foo {
    has $.bar;
    submethod BUILD(:$bar!) { $!bar = $bar }
}
```
The better way is to use the [`is required`](https://docs.raku.org/syntax/is%20required) trait on attributes.
```raku
class Foo {
    has $.bar is required;
}
```

### Allowing a named argument without automatic accessor

By allowing a named argument in the `BUILD` signature and assigning that to the attribute in the body, you're effectively mimicing the `.` twigil in the attribute definition without havin an accessor made automatically.
```raku
class Foo {
    has $!bar;
    submethod BUILD(:$bar) { $!bar = $bar }
}
```
The better way is to use the [`is built`](https://docs.raku.org/syntax/is%20built) attribute trait:
```raku
class Foo {
    has $!bar is built;
}
```

### Making an attribute immutable

Sometimes one wants to make an attribute immutable.  This can be done in the `BUILD` method by using the binding operator [`:=`](https://docs.raku.org/routine/%3A%3D) instead of the assignment operator [`=`](https://docs.raku.org/routine/%3D%20%28item%20assignment%29):
```raku
class Foo {
    has $.bar;
    submethod BUILD(:$bar) { $!bar := $bar }
}c
```
The better way is to use the [`is built`](https://docs.raku.org/syntax/is%20built) attribute trait with the `:bind` modifier:
```raaku
class Foo {
    has $.bar is built(:bind);
}
```

## When to use TWEAK

Looking at the modules in the ecosystem, the `TWEAK` method is being used for:

- throwing an error if some complicate condition is not met
- throwing an error if conflicting arguments have been specified
- starting async workers once the object is fully fleshed
- setting up a (fast) data-structure based on the attributes supplied
- setting attributes from information supplied in a (JSON) hash
- reading / parsing a file into additional attributes
- inform a logger when an object has been created

You typically should **not** use a `TWEAK` method if you can achieve the same effect with an extensive specification of the attributes.  First of all the declarative manner in which these are specified is easier to comprehend.  Secondly, the processing of the named arguments and their specificaton is highly optimized.  Compare:
```
$ raku -e 'class A { has $.a = 42 }; A.new for ^1000000; say now - ENTER now'
0.080668357
$ raku -e 'class A { has $.a; method TWEAK(:$a) { $!a = $a // 42 }; A.new for ^1000000; say now - ENTER now'
0.122443318
```
which shows that using proper attribute declarations is at least 30% faster.  Well in this case.  YMMV.

If you're more comfortable with using a `TWEAK` method, then please do.  There's more than one way to do it!

## submethod vs method

A [`submethod`](https://docs.raku.org/syntax/Submethods) is a special type of public method, but which is **not** inherited by subclasses.  `TWEAK` and `BUILD` methods should be made `submethod`s, because otherwise they will get can get executed more than once if they are in a base class and the inheriting class does **not** specify its own `TWEAK` or `BUILD` method.
```raku
class A {
    method TWEAK() { say "A" }
}
class B is A { }
B.new;
```
will show:
```
A
A
```
because class "B" inherited the `TWEAK` method from "A".  And at object initialization, the `BUILDPLAN` of each class in the class hierarchy is executed.

## Conclusion

This concludes the eight episode of cases of UPPER language elements in the Raku Programming Language, the first discussion interface methods.  In this episode the `TWEAK` and `BUILD` methods, with a supporting role for the BUILDPLAN module.

Stay tuned for the next episode!
