On Multi-dispatch in Perl 6
===========================

This is the tenth article in
[a series of articles](https://opensource.com/users/lizmat) about migrating
code from Perl 5 to Perl 6.  In this article we'll be looking at the
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

Normal Dispatch
---------------

Multiple Dispatch
-----------------


Summary
=======
