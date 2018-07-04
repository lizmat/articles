Migrating Perl 5 code to Perl 6
===============================

In the coming weeks I will be writing a number of articles about issues you
may encounter if you are a programmer who is programming / has programmed
in Perl 5, and you are interested converting Perl 5 code to Perl 6.  The
[Perl 6 documentation](https://docs.perl6.org/) already contains most, if
not all
[documentation that you need](https://docs.perl6.org/language/5to6-overview).
But as documentation goes, the focus 
and associated documents.  In these articles I will try to go a little more
in-depth about specific issues.

How is Perl 6 anyway?
---------------------
Since its first official release in December 2015, Rakudo Perl 6 has seen
an order of magnitude speed improvement, and quite a few bug fixes (more
than 14000 commits in total).  Seven books about Perl 6 have been published
so far, with the "Learning Perl 6" by brian d foy to be published shortly by
O'Reilly (re-worked from the seminal "Learning Perl" which so many people know
and love).

The user distribution "[Rakudo Star](https://rakudo.org/files)" is on a
three-monthly release cycle, and more than 1100 modules are now available in
the [Perl 6 ecosystem](https://modules.perl6.org).

Perl 5 code can be seamlessly integrated with Perl 6 using the
[Inline::Perl5](http://modules.perl6.org/dist/Inline::Perl5:cpan:NINE) module,
although one could consider this cheating, as it will embed a Perl 5
interpreter.  And therefore continues to have a dependency on the `perl` (5)
runtime.

Summary
-------
