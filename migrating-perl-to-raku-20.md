# Multi-Dispatch
This blog post introduces a Raku feature that is central to the Raku philosophy: ["Multiple Dispatch"](https://en.wikipedia.org/wiki/Multiple_dispatch), or short "multi-dispatch".

Perl does not have anything like that in core.  Several attempts have been made in module space, such as `Sub::Multi` and `Logic`, but these appear to have been abandoned long ago.

Even though you can program Raku without using multi-dispatch, you are missing out on a lot of programming ease.  So I think it’s important for someone coming from Perl to understand this feature thoroughly.

## There can be more than One
Whereas in Perl there can only be one subroutine with a given name, in Raku there can be more than one, provided you prefix the definition with the `multi` keyword.  But how will the right routine be selected then?

Even though the name of the subroutine may be identical, the signature of the subroutine should be different.  Depending on the arguments given, the right subroutine will then be selected for execution.

A simple example:
```
# Raku
# limited to integers
multi sub frobnicate (Int:D $value) {
    say "got integer: $value";
}
# not limited to a defined integer
multi sub frobnicate ($something) {
    say "got something: $something";
}
frobnicate(42);          # got integer: 42
frobnicate(Date.today);  # got something: 2023-08-06
frobnicate("foo");       # got something: foo
```
Now compare this to (almost) the same logic in Perl:
```
# Perl
sub frobnicate {
    my ($value) = @_;
    if $value =~ /^\d+$/ {  # or any other way to recognise an integer
        say "got integer: $value";
    }
    else {
        say "got something: $value";
    }
}
frobnicate(42);     # got integer: 42
frobnicate("foo");  # got something: foo
```
For this simple case, there is not a lot of difference in logic.  However, the multi-dispatch approach allows one to focus on a single case much easier, rather than winding up in an `if` / `elsif` / `else` maze of conditionals.

More importantly, if you separate the handling of different types of objects (in this case `Int` versus the rest) in separate routines, then this allows additional multi-subroutines to be added by importing them from installed modules (either from the ecosystem, or from your personal project).

Assume we have a module that handles frobnication of `Date` objects:
```
# Raku
module DateFrobnication {
    multi sub frobnicate (Date:D $date) is export {
        say "got date: $date";
    }
}
```
Note that the sub has been marked with [`is export`](https://docs.raku.org/routine/is%20export) so that it will be exported whenever the `DateFrobnication` module is loaded.

Import that module in our original example:
```
# Raku
use DateFrobnication;
multi sub frobnicate (Int:D $value) {
    say "got integer: $value";
}
# not limited to a defined integer
multi sub frobnicate ($something) {
    say "got something: $something";
}
frobnicate(42);          # got integer: 42
frobnicate(Date.today);  # got date: 2023-08-06
frobnicate("foo");       # got something: foo
```
Note that the case where the argument was a [`Date`](https://docs.raku.org/type/Date) object, is now handled by the `frobnicate` subroutine that was imported.

This would be impossible in Perl, as that would require altering the contents of the `frobnicate` subroutine at runtime.

## Also for Methods
The `multi` keyword can also be used in `method` definitions:
```
# Raku
class Foo {
    multi method frobnicate (Int:D $value) {
        say "got integer: $value";
    }
    multi method frobnicate ($something) {
        say "got something: $something";
    }
}
Foo.frobnicate(42);          # got integer: 42
Foo.frobnicate(Date.today);  # got something: 2023-08-06
Foo.frobnicate("foo");       # got something: foo
```
And whether the invocant is an instance (or not) can *also* be used in a signature:
```
# Raku
class Bar {    # note extra colon ↓
    multi method frobnicate (Bar:U:) {
        say “called on type object”;
    }
               # note extra colon ↓
    multi method frobnicate (Bar:D:) {
        say "called on object instance";
    }
}
Bar.frobnicate;      # called on type object
Bar.new.frobnicate;  # called on object instance
```
Note that the trailing colon indicates invocant, and is therefore called the "invocant marker".  Without it, it would be considered a type specification of a positional parameter.

Almost all of the commands in Raku are defined in terms of specially named multi subs.  An example: this is how the “ print “ statement in Raku is basically implemented:
```
# Raku
multi sub print (Str:D $x) {
    $*OUT.print($x)
}
multi sub print($x) {
    print($x.Str)
}
```
If you pass an object of type `Str` to `print`, then it will be printed verbatim.

If you pass anything else than a `Str` object, then it will first be converted to a `Str` object (by calling the `Str` method on it), and then passed to the `print` sub.

Note that the latter is **not** an infinite loop, because it will then be dispatched to the first candidate.

## Summary
Multi-dispatch is a very central feature to the Raku Programming Language.

For example, almost all of the operators in Raku are defined in terms of specially named multi subs.  Many methods that you can call on core objects, are also implemented as multi-methods.  Understanding the power of multi-dispatch, therefore adds a very powerful tool to your toolset.
