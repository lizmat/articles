Signatures in Perl 6
====================

The third installment of the
[Migrating Perl 5 code to Perl 6](5to6-introduction.md) series.  The first
installment was about
[garbage collection and how timely destruction works in Perl 6](5to6-finalizing.md),
the second was about [how references were replaced by the use of containers in Perl 6](5to6-containers.md).

In this post I will be focussing on (subroutine) signatures in Perl 6
and how they differ from the situation in Perl 5.

Experimental signatures in Perl 5
---------------------------------
If you're migrating Perl 5 code to Perl 6, then most likely you are not
using [the experimental signature feature](https://metacpan.org/pod/distribution/perl/pod/perlsub.pod#Signatures) that has become available in Perl 5 since
5.20, or any of the longer existing CPAN modules like
[signatures](https://metacpan.org/pod/signatures),
[Function::Parameters](https://metacpan.org/pod/Function::Parameters) or
any of the other Perl 5 modules on CPAN that
[have "signature" in their name](https://metacpan.org/search?q=signature).

Also, in my experience,
[prototypes](https://metacpan.org/pod/perlsub#Prototypes) generally have had
little usage in most Perl programs out in the world (aka
[DarkPan](http://modernperlbooks.com/mt/2009/02/the-darkpan-dependency-management-and-support-problem.html).
I will therefore only compare the Perl 6 functionality with the most common
usage of Perl 5 argument passing.

Summary
-------
Perl 6 has a way of describing how parameters should be taken and/or coerced,
and allows for subroutines (and methods) to have the same name but a different
set of acceptable parameters (Signature).  The value of the arguments is used
by the Perl 6 runtime to select the subroutine (method) to actually call
([multiple dispatch](https://en.wikipedia.org/wiki/Multiple_dispatch)).
