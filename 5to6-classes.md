Making Perl more Classy
=======================

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
`Point` class that has 2 immutable attributes: `x` and `y` and a constructor
that takes named parameters:

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

After this, creating a `Point` object is remarkably similar between Perl 5 and
Perl 6:

    # Perl 5
    my $point = Point->new( x => 42, y = 666 );

    # Perl 6
    my $point = Point.new( x => 42, y => 666 );

The only difference being the use of `.` to call a method instead of `->`.
Perhaps, more importantly, recent benchmarks have shown that Perl 6 is
actually faster at creation and accessing objects than Perl 5.

Error checking
--------------
In an ideal world, all parameters to methods will always be correctly
specified.  Unfortunately, we don't live in an ideal world, so it is wise
to add some error checking to your object creation.  Suppose we want to make
sure that both `x` and `y` are specified, and that they're integer values.

    # Perl 5
    package Point {
        sub new {
            my ($class,%args) = @_;

            die "The attribute 'x' is required" unless exists $args{x};
            die "The attribute 'y' is required" unless exists $args{y};
            die "Type check failed on 'x'" unless $args{x} =~ /^\d+\z/;
            die "Type check failed on 'y'" unless $args{y} =~ /^\d+\z/;

            bless \%args, $class
        }
        sub x { shift->{x} }
        sub y { shift->{y} }
    }

That's quite a bit of extra boilerplate.  Of course, you can abstract that
in a subroutine of your own:

    # Perl 5
    package Point {
        sub is_valid {
            my $args = shift;
            for (@_) {
                die "The attribute '$_' is required" unless exists $args->{$_};
                die "Type check failed on '$_'" unless $args{$_} =~ /^\d+\z/;
            }
            1
        }
        sub new {
            my ($class,%args) = @_;
            bless \%args, $class if is_valid(\%args,'x','y')
        }
        sub x { shift->{x} }
        sub y { shift->{y} }
    }

Or you can use one of the many parameter validation modules on CPAN, such
as [Params::Validate](https://metacpan.org/pod/Params::Validate#DESCRIPTION).

In Perl 6 however, all of that is built-in.  The `is required` attribute trait
indicates that an attribute must be specified.  And specifying a type (in this
case `Int`) will automatically throw a type check exception if the provided
value is not of the right type.

    # Perl 6
    class Point {
        has Int $.x is required;
        has Int $.y is required;
    }



Summary
=======
