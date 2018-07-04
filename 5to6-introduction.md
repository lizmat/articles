Migrating Perl 5 code to Perl 6
===============================

In the coming weeks I will be writing a number of articles about issues you
may encounter if you are a programmer who is programming / has programmed
in Perl 5, and are taking your first steps in converting Perl 5 code to Perl 6.
Or if you are simply interested in knowing about what you might encounter
when trying to port Perl 5 programs to Perl 6.

The [Perl 6 documentation](https://docs.perl6.org/) already contains most, if
not all
[documentation that you need](https://docs.perl6.org/language/5to6-overview)
to deal with all of the issues you will be confronted with in this quest.
But as documentation goes, the focus is only on the factual differences.  In
these articles I will try to go a little more in-depth about specific issues,
give a little bit more of a hands-on experience.  Experience that I've gained
while doing quite a bit of porting Perl 5 code to Perl 6 code myself.

How is Perl 6 anyway?
---------------------
Since its first official release in December 2015, Rakudo Perl 6 has seen
an order of magnitude speed improvement, and quite a few bug fixes (more
than 14000 commits in total).  Eight books about Perl 6 have been published
since then so far, with the "Learning Perl 6" by brian d foy to be published
shortly by O'Reilly (re-worked from the seminal "Learning Perl" which so many
people have come to know and love).

The user distribution "[Rakudo Star](https://rakudo.org/files)" is on a
three-monthly release cycle, and more than 1100 modules are now available in
the [Perl 6 ecosystem](https://modules.perl6.org).  Perl 6 modules are now
uploaded to [PAUSE](https://pause.perl.org/pause/query?ACTION=pause_04about)
and distributed all over the world using the [CPAN](https://www.cpan.org).

Perl 5 code can be seamlessly integrated with Perl 6 using the
[Inline::Perl5](http://modules.perl6.org/dist/Inline::Perl5:cpan:NINE) module.
One could consider this cheating, as it will embed a Perl 5 interpreter, and
therefore continues to have a dependency on the `perl` (5) runtime.

In January 2018, I proposed a
[CPAN Butterfly Plan](https://www.perl.com/article/an-open-letter-to-the-perl-community/)
to convert modules from Perl 5 to Perl 6 as closely as possible to the original
API.  Since then, over 100 Perl 5 built-in functions are now supported on
Perl 6.

Summary
-------
Rakudo Perl 6
