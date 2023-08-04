# Migrating Perl to Raku

> These blog posts are updated versions of blog posts first published on opensource.com, which were then prepared to be included in a book, and then updated again for these blog posts.

Whether you are a programmer who is taking the first steps to convert Perl code to Raku and encountering some issues. or you're just interested in learning about what might happen if you try to port Perl programs to Raku, this series of blog posts should answer at least some of your questions.

These blog posts assume familiarity with Perl as a programmer / user.   Expect concepts such as references, reference counting, and DESTROY to be discussed. If these concepts are unknown to you, then these blogs are probably not for you.

The [Raku Programming Language documentation](https://docs.raku.org) already contains most (if not all) the documentation you need to deal with the issues you will confront in migrating Perl code to Raku. But, as documentation goes, the focus is on the factual differences.  These blogs will try to go a little more in-depth about specific issues and provide a little more hands-on information based on my experience porting quite a lot of Perl code to Raku.

## Using Perl features in Raku

Perl code can be seamlessly integrated with Raku using the [Inline::Perl5](https://raku.land/cpan:NINE/Inline::Perl5) module, making all of CPAN available to any Raku program. This could be considered cheating, as it will embed a Perl interpreter and therefore continues to have a dependency on the perl runtime executor.

But it *does* make it easy to get your Raku code running (if you need access to modules that have not yet been ported) simply by adding `:from<Perl5>` to your `use` statement, like:

    use DBI:from<Perl5>;

In January 2018, I proposed a [CPAN Butterfly Plan](https://www.perl.com/article/an-open-letter-to-the-perl-community/) to convert Perl functionality to Raku as closely as possible to the original API. I stated this as a goal because Perl (as a programming language) is so much more than syntax alone. Ask anyone what Perl's unique selling point is, and they will most likely tell you it is CPAN.

## Converting Semantics

To run Perl code natively in Raku, you also need a lot of Perl semantics. Having (optional) support for Perl semantics available in Raku lowers the conceptual threshold that Perl programmers perceive when trying to program in Raku. It's easier to feel at home!

Since the publication of the CPAN Butterfly Plan, more than [100 built-in Perl functions](https://raku.land/zef:lizmat/P5built-ins) are now supported in Raku with the same API. Many functions already exist in Raku but have slightly different semantics, e.g., `shift` in Perl magically shifts from `@_` (or `@ARGV`) if no parameter is specified; in Raku the parameter is obligatory.

More than 90 [Perl CPAN distributions](https://raku.land/tags/cpan5) have also been ported to Raku while adhering to the original Perl API. These include core modules such as [`Scalar::Util`](https://raku.land/zef:lizmat/Scalar::Util) and [`List::Util`](https://raku.land/zef:lizmat/List::Util), but also non-core modules such as [`Text::CSV`](https://raku.land/github:Tux/Text::CSV) and [`Memoize`](https://raku.land/zef:lizmat/Memoize). Distributions that are upstream on the River of CPAN are targeted to have as much effect on the ecosystem as possible.

## Learning Curve
It has been pointed out that Raku is perceived to have a steep learning curve, especially coming from Perl.  To an extent, this is true.  During development of Raku, many decisions have been made to make it easier to learn Raku.  Many concepts can be applied much more generally in Raku, then they are in Perl: there are fewer exceptions to the rules.

So this brings this book in a place of the worst of both worlds.  On the one hand, the language can be best learned by people not having any previous experience in Perl.  And this book is intended for people who do have at least some Perl knowledge.  So a lot of Raku concepts are presented in how they differ from Perl, when in many cases it would probably be easier to just start with a blank slate from the underlying Raku concepts.

Still I felt that there is a point in presenting these blogs posts in this shape to you, the reader.  But please try to understand the underlying Raku principles, rather than memorise all the differences between Perl and Raku.

## Summary
Raku is a viable approach to creating new, interactive projects. Being able to use reliable and proven Perl language components aids in lowering the threshold for developers to use Raku, and it builds towards a situation where the sum of Perl and Raku becomes greater than its parts
