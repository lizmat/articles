On Calling Subs and Typing in Perl 6
============================

This is the ninth article in
[a series of articles](https://opensource.com/users/lizmat) about migrating
code from Perl 5 to Perl 6.  In this article we'll be looking at the subtle
differences of visibility of subroutines between Perl 5 and Perl 6 and the
(gradual) typing core feature of Perl 6.

This article assumes you're familiar with
[signatures](https://docs.perl6.org/type/Signature) which were
[previously discussed](https://opensource.com/article/18/9/signatures-perl-6)
in this series of articles.

So what are the (subtle) differences of subroutine visibility between Perl 5
and Perl 6?

Visibility of subroutines
-------------------------
In Perl 5, a named subroutine will by default be visible within the scope of
the **package** in which it is defined, regardless of whether the scope in
which the definition takes place:

    # Perl 5
    {
        sub foo { "bar" }       # visible outside of this scope
    }
    say foo();
    # bar

In Perl 6, a named subroutine is always only visible within the lexical scope
in which it is defined:

    # Perl 6
    {
        sub foo() { "bar" }     # only visible in this scope
    }
    say foo();
    # ===SORRY!=== Error while compiling ...
    # Undeclared routine:
    #     foo used at line ...

> Note that the `SORRY!` in the Perl 6 error message  means that the
> subroutine `foo` can not be found at **compile** time.  This is a
> terribly useful feature that helps against typos in subroutine names
> when writing invocations of the subroutine.

You could consider subroutine definitions in Perl 6 to always have a `my`
in front, similar to defining lexical variables.  Perl 5 also has a
(previously experimental)
[lexical subroutine](https://perldoc.pl/perlsub#Lexical-Subroutines)
feature, which needs to be specifically activated in versions lower than
Perl 5.26.

    # Perl 5.18 or higher
    no warnings 'experimental::lexical_subs';
    use feature 'lexical_subs';
    {
        my sub foo { "bar" }   # limit visibility to this scope
    }
    say foo();
    # Undefined subroutine &main::foo called at ...

It is possible in both Perl 5 and Perl 6 to prefix the subroutine definition
with an `our` scope indicator, but the result is subtly different: in Perl 5
this makes the subroutine visible outside of the scope, but not so in Perl 6.
In Perl 6 lookups of subroutines are **always** lexical: the use of `our` on
subroutine declarations (regardless of scope) allows that subroutine to be
called from outside of the namespace in which it is defined:

    # Perl 6
    module Foo {
        our sub bar() { "baz" }  # make sub visible from the outside
    }
    say Foo::bar();
    # baz

Which would fail without the `our`.  In Perl 5, **any** subroutine can be
called from outside of the namespace it is defined in:

    # Perl 5
    package Foo {
        sub bar { "baz" }     # always visible from the outside
    }
    say Foo::bar();
    # baz

Subroutines that are intended to be "private" in Perl 5 usually have a name
that starts with an underscore.  But that doesn't stop them from being called
from the outside.

The `our` on a subroutine definition in Perl 6 also indicates that the
subroutine in question will be exported if it is part of a module being
loaded.  But more on that in a future article about the creation of modules
and module loading.

Calling a subroutine
--------------------
When you call a subroutine in a Perl 5 without subroutine signatures enabled,
it will call the subroutine if it exists (determined at runtime) and pass the
parameters into `@_` inside the subroutine.  Whatever happens to the parameters
inside the subroutine, is entirely up to the subroutine (as
[discussed in a previous article](https://opensource.com/article/18/9/signatures-perl-6)).

Calling a subroutine in Perl 6 performs additional checks to see whether the
arguments passed to the subroutine match the parameters the subroutine expects
before it actually calls the code of the subroutine.  And it actually tries to
do this as early as possible.  If Perl 6 can determine at compile time whether
a call to a subroutine will never succeed, it will tell you at compile time:

    # Perl 6
    sub foo() { "bar" }    # subroutine not taking any parameters
    say foo(42);           # try calling it with one argument
    # ===SORRY!=== Error while compiling ...
    # Calling foo(Int) will never work with declared signature ()

Note that the error message mentions the type of value (`Int`) that is being
passed as an argument.  And in this case calling the subroutine will fail
because the subroutine doesn't accept any argument being passed to it
("declared signature ()").

Additional Signature Features
-----------------------------
Apart from specifying positional and named parameters in a signature, you can
**also** specify what type these parameters should be.  And if the type of
the parameter does not smartmatch with the type of the argument, it will be
rejected.  In this example, the subroutine expects a single `Str` argument:

    # Perl 6
    sub foo(Str $who) { "Hello $who" }  # subroutine taking a Str parameter
    say foo(42);                        # try calling it with an integer
    # ===SORRY!=== Error while compiling ...
    # Calling foo(Int) will never work with declared signature (Str $who)

So not only does it check the number of required parameters, it **also** checks
the type.  Unfortunately, it is not always possible to reliably see this at
compilation time.  But there's always the check done at runtime when binding
the argument to the parameter:

    # Perl 6
    sub foo(Str $who) { "Hello $who" }  # subroutine taking a Str parameter
    my $answer = 42;
    say foo($answer);                   # try calling it with a variable
    # Type check failed in binding to parameter '$who'; expected Str but got Int (42)
    #   in sub foo at ...

However, if Perl 6 knows the type of variable that is being passed to the
subroutine, it **can** already determine at compile time that the call will
never work:

    # Perl 6
    sub foo(Str $who) { "Hello $who" }  # subroutine taking a Str parameter
    my Int $answer = 42;
    say foo($answer);                   # try calling it with an Int variable
    # ===SORRY!=== Error while compiling ...
    # Calling foo(Int) will never work with declared signature (Str $who)

It should therefore be clear that it helps you to use typing in your
variables and parameters so that Perl 6 can help you find problems quicker!

Gradual Typing
--------------
What you just witnessed is usually described as
[gradual typing](https://en.wikipedia.org/wiki/Gradual_typing).  Perl 6 always
performs type checks at runtime (`dynamic typing`).  But as shown, if it can
determine at compile time that something will not work, it will tell you so.
Which is usually described as `static typing`.

If you're coming from Perl 5 and have experience with using
[Moose](https://metacpan.org/pod/Moose) and specifically
[MooseX::Types](https://metacpan.org/pod/MooseX::Types), you may worry about
performance implications of adding type information to your code.  This worry
is unwarranted for in Perl 6, as type checks **always** occur in Perl 6 with
every assignment to a variable or binding of a parameter.  That is because
if you do not specify a type in Perl 6, it will implicitely assume the `Any`
type, that smartmatches with (almost) everything in Perl 6.  So if you're
writing:

    # Perl 6
    my $foo = 42

You have in fact written:

    # Perl 6
    my Any $foo = 42;

And the same goes for parameters to a subroutine:

    # Perl 6
    sub foo($bar) { ... }

Is in fact:

    # Perl 6
    sub foo(Any $bar) { ... }

In fact, adding type information not only helps with finding errors in your
program, it also allows the optimizer to make better informed decisions on
what it can optimize, and how to best optimize it.

Defined or not
--------------
If you specify a variable in Perl 5 and then not assign it, it contains the
undefined value (aka `undef`):

    # Perl 5
    my $foo;
    say defined($foo) ? "defined" : "NOT defined";
    # NOT defined

This is not much different in Perl 6:

    # Perl 6
    my $foo;
    say defined($foo) ?? "defined" !! "NOT defined";
    # NOT defined

So you can specify which types of values are acceptable in a variable and as
a parameter.  But what happens if you don't assign such a variable?

    # Perl 6
    my Int $foo;
    say defined($foo) ?? "defined" !! "NOT defined";
    # NOT defined

The value inside such a variable is still not defined, just as in Perl 5.
However, if you just want to show the contents of such a variable, it is
**not** `undef` like it would be in Perl 5:

    # Perl 6
    my Int $foo;
    say $foo;
    # (Int)

What you see there is the representation of a `type object` in Perl 6.  Unlike
Perl 5, Perl 6 has a multitude of typed `undef`s.  Each class that is defined,
or which you define yourself, **is** a type object.

    # Perl 6
    class Foo { }
    say defined(Foo) ?? "defined" !! "NOT defined";
    # NOT defined

If you however instantiate a type object, usually with `.new`, it becomes
a defined object as to be expected:

    # Perl 6
    class Foo { }
    say defined(Foo.new) ?? "defined" !! "NOT defined";
    # defined

Type Smileys
------------
If you specify a constraint on a parameter in a subroutine, you can also
indicate whether you want a defined value of that type or not:

    # Perl 6
    sub foo(Int:D $bar) { ... }

The `:D` combined with the `Int` indicates that you want a `D`efined value
of the `Int` type.  Because of emoji's often using `:D` for big smiles, this
decoration on the type is called a "type smiley".  So what happens if you
pass an undefined value to such a subroutine?

    # Perl 6
    sub foo(Int:D $bar) { ... }   # only accept instances of Int
    foo(Int);                     # call with a type object
    # Parameter '$bar' of routine 'foo' must be an object instance of
    # type 'Int', not a type object of type 'Int'.  Did you forget a '.new'?

Careful readers may have realized that this should have been a compile time
error.  But alas, it isn't (yet).  Although error messages are known to be
pretty awesome in Perl 6, there is still a lot of work to be done to make
them even better (and more timely in this case).

You can also use the `:D` type smiley on variable definitions.  This will
ensure that you provide an initialization for that variable:

    # Perl 6
    my Int:D $foo;                # missing initialization
    # ===SORRY!=== Error while compiling ...
    # Variable definition of type Int:D requires an initializer

Other type smileys are `:U` (for **un**defined) and `:_` (for don't care,
which is the default).  So:

    # Perl 6
    sub foo(Int:U $bar) { ... }   # only accept Int type object
    foo(42);                      # call with an instance of Int
    # Parameter '$bar' of routine 'foo' must be a type object of type 'Int',
    # not an object instance of type 'Int'.  Did you forget a 'multi'?

Hmmm... what's this **multi** that may seem to be forgotten?  Well, that's
for the next article in this series!

Summary
=======
Subroutines in Perl 6 are by default only visible in the lexical scope where
they are defined.  Even prefixing `our` will not make them visible outside of
that lexical scope, but it **does** allow a subroutine to be called from
outside the scope with its full package name (`Foo::bar()` for a subroutine
`bar` in a package `bar`).

Perl 6 allows you to use gradual typing to ensure the validity of arguments
to subroutines, or assignments to variables.  This does **not** incur any
extra runtime costs.  Adding typing to your code even allows the compiler
to catch errors at compile time, and it allows the optimizer to make better
decisions about optimizing during runtime.
