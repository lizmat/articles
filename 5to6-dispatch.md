On Subs and Dispatch in Perl 6
==============================

This is the ninth article in
[a series of articles](https://opensource.com/users/lizmat) about migrating
code from Perl 5 to Perl 6.  In this article we'll be looking at the subtle
differences of visibility of subroutines between Perl 5 and Perl 6 and the
multi-dispatch core feature of Perl 6.

Perl 5 doesn't have multi-dispatch as a core feature, but it does have two
modules on CPAN that provide a subset of the functionality of Perl 6
multi-dispatch:
[Class::Multimethods](https://metacpan.org/pod/Class::Multimethods) by Damian
Conway (from 2000, which clearly provided inspiration for the multi-dispatch
feature of Perl 6), and
[MooseX::MultiMethods](https://metacpan.org/pod/MooseX::MultiMethods) by
Florian Ragwitz (from 2009, which clearly has been inspired by multi-dispatch
of Perl 6).  The cross-pollination between Perl 5 and Perl 6 is very clear for
these features.

This article assumes you're familiar with
[signatures](https://docs.perl6.org/type/Signature) which were
[previously discussed](https://opensource.com/article/18/9/signatures-perl-6)
in this series of articles.

Visibility of subroutines
-------------------------
In Perl 5, a named subroutine will always be visible within the scope of
the package in which it is defined, regardless of whether the scope in which
the definition takes place:

    # Perl 5
    {
        sub foo { "bar" }       # visible outside of this scope
    }
    say foo();
    # bar

In Perl 6, by default a named subroutine is only visible within the lexical
scope in which it is defined:

    # Perl 6
    {
        sub foo() { "bar" }     # only visible in this scope
    }
    say foo();
    # ===SORRY!=== Error while compiling ...
    # Undeclared routine:
    #     foo used at line ...

This is because when you define a subroutine without a scope indicator, it
acts like it has a `my` in front, just like you do when defining lexical
variables.  Perl 5 also has a (previously experimental)
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

> Note that the `SORRY!` in the Perl 6 error message  means that the
> subroutine `foo` can not be found at **compile** time.  This is a
> terribly useful feature that helps against typos in subroutine names
> when writing invocations of the subroutine.

Normal Dispatch
---------------



Summary
=======
