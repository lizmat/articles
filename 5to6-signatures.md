Migrating Perl 5 code to Perl 6
===============================

In this post I will be focussing more on (subroutine) signatures in Perl 6
and how they differ from the situation in Perl 5.

Part 2 - Signatures
===================

Conclusion
----------
Older versions of Perl 5 only have a so-called `prototype` to a subroutine.
These could be seen as a sort of coercer in the Perl 6 context.  Perl 6 has
a way of describing how parameters to *any* block should be taken and/or
coerced, and allows for subroutines (and methods) to have the same name
but a different set of acceptable parameters (signature).  The value of
the arguments is used by the Perl 6 runtime to select the correct subroutine
(method) to actually call (multi-method dispatch).
