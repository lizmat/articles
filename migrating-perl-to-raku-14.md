# Classes (Part 1 of 2)
The following 2 blogs posts look at how to create classes (objects) in Raku and how it differs from Perl.

## On Moose
Perl has a very basic form of object orientation, which you could argue has been bolted on as an afterthought. Several attempts have been made to improve that situation, most notably `Moose`, which "is based in large part on the Raku object system, as well as drawing on the best ideas from CLOS, Smalltalk, and many other languages." And, in turn, the Raku object creation logic has taken a few lessons from Moose.

Moose has inspired a number of other modern object systems in Perl, most notably `Moo` and Mouse`. Before you start a new project in Perl, it is recommend to read `Modern Perl` book; among other things, it describes how to use Moose to create classes/objects.

For simplicity, this chapter will describe the general differences between basic Perl and basic Raku object creation.

## How to make a 'Point'
A picture is worth more than a thousand words. So, let's start with defining a `Point` class with two immutable attributes, "x" and "y", and a `new` constructor that takes named arguments.

Here's how it would look in Perl:
```
# Perl
package Point {
    sub new {
        my $class = shift;
        my %args  = @_;  # maps remaining args as key / value into hash
        bless \%args, $class
    }
    sub x { shift->{x} }
    sub y { shift->{y} }
}
```
And in Raku:
```
# Raku
class Point {
    has $.x;
    has $.y;
}
```
As you can see, the Raku syntax is much more declarative; there is no need to write code to have a new method, nor is code needed to create the accessors for "x" and "y". Also note that instead of `package, you need to specify `class` in Raku.

After this, creating a Point object is remarkably similar in Perl and Raku:
```
# Perl
my $point = Point->new( x => 42, y = 666 );
```
```
# Raku
my $point = Point.new( x => 42, y => 666 );
```
As we’ve seen earlier, the only difference is that Raku uses a period to call a method instead of `->` (hyphen + greater-than symbol).

## Error checking
In an ideal world, all parameters to methods would always be correctly specified. Unfortunately, we don't live in an ideal world, so it is wise to add error checking to your object creation.

Suppose you want to make sure that both "x" and "y" are specified and are integer values. In Perl, you could do it like this:
```
# Perl
package Point {
    sub new {
        my ( $class, %args ) = @_;
        die "The attribute 'x' is required” unless exists $args{x};
        die "The attribute 'y' is required” unless exists $args{y};
        die "Type check failed on ‘x'" unless $args{x} =~ /^-?\d+\z/;
        die "Type check failed on ‘y'" unless $args{y} =~ /^-?\d+\z/;
        bless \%args, $class;
    }
    sub x { shift->{x} }
    sub y { shift->{y} }
}
```
Pardon the `/^-?\d+\z/` line noise. This is a regular expression checking for an optional (`?`) hyphen (`-`) at the start of a string (`^`) consisting of one or more decimal digits (`\d+`) until the end of the string (`\z`).

That's quite a bit of extra boilerplate. Of course, you can abstract that into an "is_valid" subroutine, like this:
```
# Perl
sub are_valid {
    my $args = shift;
    for (@_) {        # loop over all keys specified
        die "The attribute '$_' is required"
          unless exists $args->{$_};
        die "Type check failed on '$_'"
          unless $args->{$_} =~ /^-?\d+\z/;
    }
    1;  # a true value
}
```
Or you can use one of the many parameter-validation modules on CPAN, such as `Params::Validate`. In any case, your code would look something like this:
```
# Perl
package Point {
    sub new {
        my ( $class, %args ) = @_;
        bless \%args, $class if are_valid(\%args,'x','y');
    }
    sub x { shift->{x} }
    sub y { shift->{y} }
}
Point->new( x => 42, y => 666 );     # ok
Point->new( x => 42 );               # 'y' missing
Point->new( x => "foo", y => 666 );  # 'x' is not an int
```
If you use Moose, your code would look something like this:
# Perl
package Point;
use Moose;
has 'x' => ( is => 'ro', isa => 'Int', required => 1);
has 'y' => ( is => 'ro', isa => 'Int', required => 1);
no Moose;
__PACKAGE__->meta->make_immutable;
Point->new( x => 42, y => 666 );     # ok
Point->new( x => 42 );               # 'y' missing
Point->new( x => "foo", y => 666 );  # 'x' is not an int
```
Note that with an object system like Moose, you do not need to create a `new` subroutine, just like in Raku where you do not need to create a `new` method.

In Raku this is all is built-in. The `is required` attribute trait indicates that an attribute must be specified. And specifying a type (e.g., `Int`) automatically throws a type-check exception if the provided value is not an acceptable type:
```
# Raku
class Point {
    has Int $.x is required;
    has Int $.y is required;
}
Point.new( x => 42, y => 666 );     # ok
Point.new( x => 42 );               # 'y' missing
Point.new( x => "foo", y => 666 );  # 'x' is not an int
```

## Providing defaults
Alternately, you might want to make the attributes optional and have them initialised to 0 if they are not specified. In Perl, that could look like this:
```
# Perl
package Point {
    sub new {
        my ( $class, %args ) = @_;
        $args{x} //= 0;
        $args{y} //= 0; # initialize to 0 is not given
        bless \%args, $class if is_valid( \%args, 'x', 'y' );
    }
    sub x { shift->{x} }
    sub y { shift->{y} }
}
```
Although technically  `//=` is not the same as a check for existence of a key in the hash, in practice this is a very common Perl idiom to check for named arguments being specified.

In Raku, you would add an assignment with the default value to each attribute declaration:
```
# Raku
class Point {
    has Int $.x = 0;  # initialise to 0 if not given
    has Int $.y = 0;
}
```

## Just Saying Again
One advantage of objects in Raku over objects in Perl, is that they come with a useful stringification.  Compare the output in Perl:
```
# Perl
say Point->new(x => 10, y => 20);  # Point=HASH(0x7fcdfe003418)
```
with the output in Raku for the above defined `Point` classes:
```
# Raku
say Point.new(x => 10, y => 20);   # Point.new(x => 10, y => 20)
```
Which can be **very** helpful in debugging.

## Summary
In this blog post the difference between the classic core Perl approach to creating classes, and the more descriptive way of the Raku Programming Language were highlighted.
