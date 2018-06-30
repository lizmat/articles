Migrating Perl 5 code to Perl 6
===============================

In the coming weeks I will be writing a number of articles about issues you
may encounter if you are a programmer who is programming / has programmed
in Perl 5, when you are doing your first steps in Perl 6.  Most, if not all,
are (already documented)[https://docs.perl6.org/language/5to6-nutshell] and
associated documents.  In these articles I will try to go a little more
in-depth about specific issues.

Part 1 - Garbage Collection
===========================
There is *no* timely destruction of objects in Perl 6.

