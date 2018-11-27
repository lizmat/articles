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
of Perl 6.  The cross-pollination between Perl 5 and Perl 6 is very clear for
these features.

Visibility of subroutines
-------------------------
In Perl 5, a named subroutine will always be visible within the scope of
the package in which it is defined, regardless of whether the scope in which
the definition takes place:

    # Perl 5
    {
        sub foo { "bar" }
    }
    say foo;
    # bar

In Perl 6, by default a named subroutine is only visible within the lexical
scope in which it is defined:

    # Perl 6
    {
        sub foo() { "bar" }     # implicit "my"
    }
    say foo;
    # ===SORRY!=== Error while compiling ...
    # Undeclared routine:
    #     foo used at line ...

Note that Perl 6 sees that the subroutine "foo" can not be found at **compile**
time.  This is because when you define a subroutine without a scope indicator,
it defaults to `my` in Perl 6.

    # Perl 6
    {
        our sub foo() { "bar" }
    }
    say foo;
    # bar

Normal Dispatch
---------------



Summary
=======
