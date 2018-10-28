How to use modules in Perl 6
============================

This is the seventh article in
[a series of articles](https://opensource.com/users/lizmat) about migrating
code from Perl 5 to Perl 6.  In this article we'll be looking at how to
create classes (objects) in Perl 6, and how that differs from Perl 5.

Perl 5 has a very basic form of object orientation, which you could argue
has been bolted on as an afterthought.  Several attempts have been made to
improve that situation, most notably
[Moose](https://metacpan.org/pod/distribution/Moose/lib/Moose/Manual.pod)
which states:

> Moose is based in large part on the Perl 6 object system, as well as
> drawing on the best ideas from CLOS, Smalltalk, and many other languages.

`Moose`, in its turn has inspired more modern object systems in Perl 5,
most notably [Moo](https://metacpan.org/pod/Moo#DESCRIPTION) and
[Mouse](https://metacpan.org/pod/Mouse#DESCRIPTION).  More generally, if
you're planning on starting a new project in Perl 5, reading
[Modern Perl](http://onyxneon.com/books/modern_perl/modern_perl_2016_a4.pdf)
by *chromatic* is recommended: among many other things, it describes how
to use `Moose` to create classes / objects.

For simplicity, this article will only describe the differences between
basic Perl 5 and Perl 6 object creation.

How to make a Point
-------------------
A picture is better than a thousand words.  So let's start with defining a
`Point` class that has 2 immutable attributes: `x` and `y`:

    # Perl 5
    package Point {
        sub new {
            my ($class,%args) = @_;
            bless \%args, $class
        }
        sub x { shift->{x} }
        sub y { shift->{y} }
    }

In Perl 6 this would look like:

    # Perl 6
    class Point {
        has $.x;
        has $.y;
    }

As you can see, the Perl 6 syntax is much more declarative: there is no need
to write code to have a `new` method, nor is there code needed to create the
accessors for `x` and `y`.  Also note that instead of `package`, one needs to
specify `class` in Perl 6.

Summary
=======
