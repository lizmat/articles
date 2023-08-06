# On Subs And Typing
This blog post examines the subtle differences in visibility of subroutines between Perl and Raku and the (gradual) typing core feature of the Raku Programming Language.  It assumes you're familiar with signatures — if you're not, read the "Subroutine Signatures" blog post before you continue.

## Visibility of subroutines
In Perl, a named subroutine will by default be visible within the scope of the package in which it is defined, regardless of the scope where the definition takes place:
```
# Perl
{
    sub foo { "bar" }       # visible outside of this scope
}
say foo(); # bar
```
In Raku, a named subroutine is visible only within the lexical scope in which it is defined:
```
# Raku
{
    sub foo () { "bar" }     # only visible in this scope
}
say foo();
# ===SORRY!=== Error while compiling …
# Undeclared routine:
#     foo used at line …
```
Note that "SORRY!" in the Raku error message means that the subroutine `foo` can't be found at **compile** time. This is a very useful feature that helps prevent typos in subroutine names when writing invocations of the subroutine.

You could consider subroutine definitions in Raku to always have a `my` in front, similar to defining lexical variables.  Perl also has a (previously experimental) lexical subroutine feature, which has to be specifically activated in versions lower than Perl 5.26:
```
# Perl 5.18 or higher
no warnings 'experimental::lexical_subs';
use feature 'lexical_subs';
{
    my sub foo { "bar" }   # limit visibility to this scope
}
say foo();
# Undefined subroutine &main::foo called at …
```
It is possible in both Perl and Raku to prefix the subroutine definition with an [`our`](https://docs.raku.org/language/variables#The_our_declarator) scope indicator, but the result is subtly different.

In Perl, this makes the subroutine visible outside the scope, but this isn't the case in Raku.

In Raku, lookups of subroutines are **always** lexical: the use of `our` on subroutine declarations (regardless of scope) allows the subroutine to be called from outside the namespace in which it is defined:
```
# Raku
module Foo {
    our sub bar () { "baz" }  # make sub visible outside
}
say Foo::bar();# baz
```
This would fail without the `our`. In Perl, any subroutine can be called from outside the namespace where it is defined:
```
# Perl
package Foo {
    sub bar { "baz" }     # always visible from the outside
}
say Foo::bar();# baz
```
In Perl, the names of subroutines that are intended to be "private" (i.e., called from within that scope only and not from outside) usually start with an underscore. But that won't stop them from being called from the outside.  In Raku, subroutines that are not intended to be called from the outside are simply invisible.

The `our` on a subroutine definition in Raku not only indicates that the subroutine can be called from the outside; it also indicates that it will be exported if it is part of a module being loaded.

# Calling a subroutine
When you call a subroutine in a Perl without subroutine signatures enabled, it will call the subroutine if it exists (determined at runtime) and pass the parameters into `@_` inside the subroutine. Whatever happens to the parameters inside the subroutine is entirely up to the subroutine (see the blog post about Subroutine Signatures).

When a subroutine is called in Raku, it performs additional checks to see whether the arguments passed to the subroutine match the parameters the subroutine expects before it calls the subroutine's code.

Raku tries to do this as early as possible — if it determines that a call to a subroutine will never succeed, it will tell you at compile time:
```
# Raku
sub foo () { "bar" }  # subroutine not taking any parameters
say foo(42);          # try calling it with one argument
# ===SORRY!=== Error while compiling …
# Calling foo(Int) will never work with declared signature ()
```
Note that the error message mentions the type of value (*Int*) being passed as an argument. In this case, calling the subroutine will fail because the subroutine doesn't accept any argument being passed to it (*declared signature ()*).

# Other signature features
Apart from specifying positional and named parameters in a signature, you can also specify what type these parameters should be.  If the parameter type doesn't smartmatch with the argument type, it will be rejected.

In this example, the subroutine expects a single [`Str`](https://docs.raku.org/type/Str) argument:
```
# Raku
# subroutine taking a Str parameter
sub foo (Str $who) { "Hello $who" }
# try calling it with an integer
say foo(42);                        
# ===SORRY!=== Error while compiling …
# Calling foo(Int) will never work with declared signature (Str $who)
```
It checks both the *number* of required parameters **and** the type. Unfortunately, it is not always possible to reliably see this at compilation time.

But there's still the check done at runtime when binding the argument to the parameter:
```
# Raku
sub foo (Str $who) { "Hello $who" }
# subroutine taking a Str parameter
my $answer = 42;
# try calling it with a variable
say foo($answer);
# Type check failed in binding to parameter ‘$who';
#   expected Str but got Int (42)
#   in sub foo at …
```
However, if Raku knows the type of variable being passed to the subroutine, it can determine at compile time that the call will never work:
```
# Raku
# subroutine taking a Str parameter
sub foo (Str $who) { "Hello $who" }
my Int $answer = 42;
# try calling with an Int variable
say foo($answer);
# ===SORRY!=== Error while compiling …
# Calling foo(Int) will never work with declared signature (Str $who)
```
It should be clear that using typing in your variables and parameters enables Raku to help you find problems quicker!

## Gradual typing
The above is usually called ["gradual typing"](https://en.wikipedia.org/wiki/Gradual_typing). Raku always performs type checks at runtime ("dynamic typing"). But if it can determine at compile time that something will not work, it will tell you so. This is usually called "static typing".

If you're coming from Perl and have experience with Moose (and specifically `MooseX::Types`), you may worry about the performance implications of adding type information to your code. This is not a concern in Raku, as type checks **always** occur in Raku with **every** assignment to a variable or binding of a parameter. That is because if you do not specify a type, Raku will implicitly assume the `Any` type, which smartmatches with (almost) everything in Raku.

So, if you're writing:
```
# Raku
my $foo = 42;
```
You have in fact written:
```
# Raku
my Any $foo = 42;
```
And the same goes for a parameter to a subroutine:
```
# Raku
sub foo ($bar) { ... }
```
Which is in fact:
```
# Raku
sub foo (Any $bar) { ... }
```
Adding type information not only helps find errors in your program, it also allows the optimiser to make better-informed decisions about what it can optimise and how to best optimise it.

## Defined or not
If you specify a variable in Perl but don't assign it, it contains the undefined value (aka `undef`):
```
# Perl
my $foo;
say defined($foo) ? "defined" : "NOT defined";
# NOT defined
```
This is not much different in Raku:
```
# Raku
my $foo;
say defined($foo) ?? "defined" !! "NOT defined";
# NOT defined
```
So, you can specify which types of values are acceptable in a variable and as a parameter.

But what happens if you don't assign such a variable?
```
# Raku
my Int $foo;
say defined($foo) ?? "defined" !! "NOT defined";
# NOT defined
```
The value inside such a variable is still not defined, as in Perl.

However, if you just want to show the contents of such a variable, it is **not** "undef", as it would be in Perl:
```
# Raku
my Int $foo;
say $foo;# (Int)
```
What you see is the string representation of a ["type object"](https://docs.raku.org/language/glossary#Type_objects) in Raku.  Unlike Perl, Raku has a multitude of typed "undef" values.

Each class that is defined, or which you define yourself, is a type object.
```
# Raku
class Foo { }
say defined(Foo) ?? "defined" !! "NOT defined";
# NOT defined
```
If, however, you instantiate a type object, usually with calling the `new` method, it becomes a defined object, as expected:
```
# Raku
class Foo { }
say defined(Foo.new) ?? "defined" !! "NOT defined";
# defined
```

## Type smileys
If you specify a constraint on a parameter in a subroutine, you can also indicate whether you want a defined value of that type or not:
```
# Raku
sub foo (Int:D $bar) { ... }
```
The `:D` combined with the `Int` indicates that you want a *defined* value of the `Int` type.  Because `:D` is also the emoji for a big smile, this decoration on the type is called a ["type smiley"](https://docs.raku.org/language/glossary#Type_smiley).

So what happens if you pass an undefined value to such a subroutine?
```
# Raku
# only accept instances of Int
sub foo (Int:D $bar) { ... }
# call with a type object
foo(Int);
# Parameter '$bar' of routine 'foo' must be an object
# instance of type 'Int', not a type object of type 'Int'.
# Did you forget a '.new'?
```
Careful readers may realise that this should create a compile-time error. But alas, it hasn't (yet). Although error messages are known to be pretty awesome in Raku, there is still a lot of work to make them even better (and more timely, in this case).

You can also use the `:D` type smiley on variable definitions to ensure that you provide an initialisation for that variable:
```
# Raku
my Int:D $foo;  # missing initialisation
# ===SORRY!=== Error while compiling …
# Variable definition of type Int:D
# requires an initializer
```
Other type smileys are `:U` (for undefined) and `:_` (for don't care, which is the default):
```
# Raku
# only accept Int type object
sub foo (Int:U $bar) { ... }
# call with an instance of Int
foo(42);
# Parameter '$bar' of routine 'foo' must be
# a type object of type 'Int',
# not an object instance of type 'Int'.
# Did you forget a 'multi'?
```
Hmmm… what's this `multi` that seems to be forgotten?  Please check the next blog post for more info on that!

## Summary
Subroutines in Raku are, by default, visible only in the lexical scope where they are defined. Even prefixing `our` will not make them visible outside of that lexical scope, but it does allow a subroutine to be called from outside the scope with its full package name (`Foo::bar()` for a subroutine `bar` in a package `Foo`).

Raku allows you to use gradual typing to ensure the validity of arguments to subroutines or assignments to variables. This does not incur any extra runtime costs. Adding typing to your code even allows the compiler to catch errors at compile time, and it allows the optimiser to make better decisions about optimising during runtime.
