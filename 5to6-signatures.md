Migrating Perl 5 code to Perl 6
===============================

In this post I will be focussing more on (subroutine) signatures in Perl 6
and how they differ from the situation in Perl 5.

If you're migrating Perl 5 code to Perl 6, then most likely you are not
using any of the experimental signature features that have become available
in Perl 5 since 5.20.  Also, in my experience,
[prototypes](https://metacpan.org/pod/perlsub#Prototypes) generally have had
little usage in most Perl programs out in the world (aka
[DarkPan](http://modernperlbooks.com/mt/2009/02/the-darkpan-dependency-management-and-support-problem.html).
I will therefore only compare the Perl 6 functionality with the most common
usage of Perl 5 argument passing and prototypes.

Part 3 - Signatures
===================

Summary
-------
Older versions of Perl 5 only have a so-called `prototype` to a subroutine.
These could be seen as a sort of coercer in the Perl 6 context.  Perl 6 has
a way of describing how parameters to *any* block should be taken and/or
coerced, and allows for subroutines (and methods) to have the same name
but a different set of acceptable parameters (signature).  The value of
the arguments is used by the Perl 6 runtime to select the correct subroutine
(method) to actually call (multi-method dispatch).
