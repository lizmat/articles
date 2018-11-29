On Subs and Typing in Perl 6
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


Summary
=======
