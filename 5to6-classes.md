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

Vice-versa, the Perl 6 object creation logic has taken a few lessons
learned by `Moose`.

`Moose` in its turn has inspired a number of other modern object systems in
Perl 5, most notably [Moo](https://metacpan.org/pod/Moo#DESCRIPTION) and
[Mouse](https://metacpan.org/pod/Mouse#DESCRIPTION).  More generally, if
you're planning on starting a new project in Perl 5, reading
[Modern Perl](http://onyxneon.com/books/modern_perl/modern_perl_2016_a4.pdf)
by *chromatic* is recommended: among many other things, it describes how
to use `Moose` to create classes / objects.

For simplicity, this article generally will only describe the differences
between basic Perl 5 and basic Perl 6 object creation.

How to make a Point
-------------------
A picture is better than a thousand words.  So let's start with defining a
`Point` class that has 2 immutable attributes: `x` and `y` and a constructor
that takes named parameters:

    # Perl 5
    package Point {
        sub new {
            my $class = shift;
            my %args  = @_;  # maps remaining args as key / value into hash
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

The only difference being the use of "`.`" (period) to call a method instead
of "`->`" (hyphen greater-than).

Error checking
--------------
In an ideal world, all parameters to methods will always be correctly
specified.  Unfortunately, we don't live in an ideal world, so it is wise
to add some error checking to your object creation.  Suppose we want to make
sure that both `x` and `y` are specified, and that they're integer values.
In Perl 5 one could do it like this:

    # Perl 5
    package Point {
        sub new {
            my ( $class, %args ) = @_;
            die "The attribute 'x' is required" unless exists $args{x};
            die "The attribute 'y' is required" unless exists $args{y};
            die "Type check failed on 'x'" unless $args{x} =~ /^-?\d+\z/;
            die "Type check failed on 'y'" unless $args{y} =~ /^-?\d+\z/;
            bless \%args, $class
        }
        sub x { shift->{x} }
        sub y { shift->{y} }
    }

> Pardon the `/^-?\d+\z/` line noise.  This is a regular expression checking
> for an optional (`?`) hyphen (`-`) at the start of a string (`^`) consisting
> of one or more decimal digits (`\d+`) until the end of the string `(\z)`.

That's quite a bit of extra boilerplate.  Of course, you can abstract that
into a subroutine "`is_valid`" of your own:

    # Perl 5
    sub is_valid {
        my $args = shift;
        for (@_) {        # loop over all keys specified
            die "The attribute '$_' is required" unless exists $args->{$_};
            die "Type check failed on '$_'" unless $args->{$_} =~ /^-?\d+\z/;
        }
        1
    }

Or you can use one of the many parameter validation modules on CPAN, such
as [Params::Validate](https://metacpan.org/pod/Params::Validate#DESCRIPTION).
In any case, your code would look something like:

    # Perl 5
    package Point {
        sub new {
            my ( $class, %args ) = @_;
            bless \%args, $class if is_valid(\%args,'x','y');
        }
        sub x { shift->{x} }
        sub y { shift->{y} }
    }
    Point->new( x => 42, y => 666 );     # ok
    Point->new( x => 42 );               # 'y' missing
    Point->new( x => "foo", y => 666 );  # 'x' is not an integer

Or in Perl 5 using
[Moose](https://metacpan.org/pod/distribution/Moose/lib/Moose/Manual.pod).
Note that with an object system like `Moose`, one does not need to create a
`new` subroutine, just as in Perl 6:

    # Perl 5
    package Point;
    use Moose;
    has 'x' => ( is => 'ro', isa => 'Int', required => 1);
    has 'y' => ( is => 'ro', isa => 'Int', required => 1);
    no Moose;
    __PACKAGE__->meta->make_immutable;
    Point->new( x => 42, y => 666 );     # ok
    Point->new( x => 42 );               # 'y' missing
    Point->new( x => "foo", y => 666 );  # 'x' is not an integer

In Perl 6 however, all of that is built-in.  The "`is required`" attribute
trait indicates that an attribute *must* be specified.  And specifying a
type (in this case "`Int`") will automatically throw a type check exception
if the provided value is not of an acceptable type.

    # Perl 6
    class Point {
        has Int $.x is required;
        has Int $.y is required;
    }
    Point.new( x => 42, y => 666 );     # ok
    Point.new( x => 42 );               # 'y' missing
    Point.new( x => "foo", y => 666 );  # 'x' is not an integer

Providing defaults
------------------
Alternately, you might just want to make the attributes optional and have them
be initialized to `0` if they are not specified.  In Perl 5 that could look
like this:

    # Perl 5
    package Point {
        sub new {
            my ( $class, %args ) = @_;
            $args{x} = 0 unless exists $args{x};
            $args{y} = 0 unless exists $args{y};
            bless \%args, $class if is_valid( \%args, 'x', 'y' );
        }
        sub x { shift->{x} }
        sub y { shift->{y} }
    }

In Perl 6 one would add an assignment with the default value to each attribute
declaration:

    # Perl 6
    class Point {
        has Int $.x = 0;  # initialize to 0 if not specified
        has Int $.y = 0;
    }

Providing mutators
------------------
In the class / object examples so far, the attributes of an object have been
immutable: they can not be changed by normal means once the object has been
created.

In Perl 5 there are a number of ways to create a mutator (a method on the
object to change the value of an attribute).  The simplest way is to create
a separate subroutine that will set the value in the object:

    # Perl 5
    ...
    sub set_x {
        my $object = shift;
        $object->{x} = shift;
    }

which can be shortened to:

    # Perl 5
    ...
    sub set_x { $_[0]->{x} = $_[1] }  # access elements in @_ directly

so you could use it as:

    # Perl 5
    my $point = Point->new( x => 42, y => 666 );
    $point->set_x(314);

Some people prefer to use the same subroutine name for both accessing and
mutating the attribute.  Specifying a parameter then means the subroutine
should be used as a mutator:

    # Perl 5
    ...
    sub x {
        my $object = shift;
        @_ ? $object->{x} = shift : $object->{x}
    }

which can be shortened to:

    # Perl 5
    ...
    sub x { @_ > 1 ? $_[0]->{x} = $_[1] : $_[0]->{x} }

so you could use it as:

    # Perl 5
    my $point = Point->new( x => 42, y => 666 );
    $point->x(314);

Then there is the way that is used a lot, but which depends on the
implementation detail of how objects are implemented in Perl 5.  Since an
object in Perl 5 is usually just a hash reference with benefits, one *can*
use the object as a hash reference and directly access keys in the underlying
hash.  But this breaks the encapsulation of the object, and bypasses any
additional checks that a mutator may do:

    # Perl 5
    my $point = Point->new( x => 42, y => 666 );
    $point->{x} = 314;  # change x to 314 unconditionally: dirty but fast

And then there is an "official" way of creating accessors that can also be
used as mutators using
[lvalue subroutines](https://perldoc.pl/perlsub#Lvalue-subroutines), but
which isn't used a lot in Perl 5 for various reasons.  But which *is* very
close to how mutators work in Perl 6:

    # Perl 5
    ...
    sub x : lvalue { shift->{x} }  # make "x" an lvalue sub

so you could use it as:

    # Perl 5
    my $point = Point->new( x => 42, y => 666 );
    $point->x = 314;  # just as if $point->x is a variable

In Perl 6, allowing an accessor to be used as a mutator, is also done in a
declarative way by using the `is rw` trait on the attribute declaration, just
like with the `is required` trait:

    # Perl 6
    class Point {
        has Int $.x is rw = 0;  # allowed to change, default is 0
        has Int $.y is rw = 0;
    }

Which allows you to use it like this in Perl 6:

    # Perl 6
    my $point = Point.new( x => 42, y => 666 );
    $point.x = 314;  # just as if $point.x is a variable

If you don't like the way mutators work in Perl 6, you're completely free to
create your own mutators by adding a method for them.  For example, the
`set_x` case from Perl 5 could look like this in Perl 6:

    # Perl 6
    class Point {
        has $.x;
        has $.y;
        method set_x($new) { $!x = $new }
        method set_y($new) { $!y = $new }
    }

But **wait**: what's the exclamation mark doing there in "`$!x`" **???**

The "`!`" indicates the *real* name of the attribute in the class: it gives
direct access to the attribute in the object.  Let's take a step back and see
what the so-called
[twigil](https://docs.perl6.org/language/variables#index-entry-Twigil) (aka
secondary sigil) of the attribute means.

The ! (exclamation mark) twigil
-------------------------------
In a declaration of an attribute like "`$!x`", the "`!`" means that the
attribute is *private*.  This means that you cannot access that attribute
from the outside, unless the developer of the class has provided a means to
do so.  This *also* means that it can **not** be initialized with a call to
`.new`.

A method for accessing the private attribute value can be very simple:

    # Perl 6
    class Point {
        has $!x;            # ! indicates a private attribute
        has $!y;
        method x() { $!x }  # return private attribute value
        method y() { $!y }
    }

And this is in fact pretty much what happens automatically if you declare
the attribute with the "`.`" twigil.

The . (period) twigil
---------------------
In a declaration of an attribute like "`$.x`", the "`.`" means that the
attribute is *public*.  This means that an accessor method is created for
it (much like the example above with the method for the private attribute).
This **also** means that the attribute **can** be initialized with a call to
`.new`.

If you otherwise using the attribute form "`$.x`", you are not really
referring to the attribute, but rather to its *accessor*.  It is in fact
syntactic sugar for "`self.x`".  But the "`$.x`" form has the advantage that
you can easily interpolate inside a string.  Furthermore, the accessor can be
overridden by a subclass.

    # Perl 6
    class Answer {
        has $.x = 42;
        method message() { "The answer is $.x" }  # use accessor in message
    }
    class Fake is Answer {   # subclassing is done with "is" trait
        method x() { 666 }   # override the accessor in Answer
    }
    say Answer.new.message;  # The answer is 42
    say Fake.new.message;    # The answer is 666 (even though $!x is 42)

Tweaking object creation
------------------------
Sometimes you need to perform extra checks / tweaks to an object before it is
really ready for consumption.  Without getting into the
[nitty gritty of creating objects in Perl 6](https://docs.perl6.org/language/classtut),
usually you can do all the tweaking that you need by supplying a `TWEAK`
method.  Suppose you also want to allow the value `314` to be considered as
an alternative to `666`:

    # Perl 6
    class Answer {
        has Int $.x = 42;
        submethod TWEAK() {
            $!x = 666 if $!x == 314;  # 100 x pi is also bad
        }
    }

If a class has a `TWEAK` method, then it will be called **after** all arguments
have been processed and assigned to attributes as appropriate (including
assigning any default values and any processing of traits such as "`is rw`"
and "`is required`").  Inside the method, you can do all that you want to
the attributes in the object.

> Note that the `TWEAK` method is best implemented as a so-called
> `submethod`.  A `submethod` is a special type of method that can only be
> executed on the class itself, and *not* on any subclass.  In other words,
> this method has the visibility of a `sub`routine.

Positional Parameters
---------------------
Finally, sometimes an interface to an object is so clear, that you do not
need named parameters at all,  Instead you want to use positional parameters.
In Perl 5, that would look something like this:

    # Perl 5
    package Point {
        sub new {
            my ( $class, $x, $y ) = @_;
            bless { x => $x, y => $y }, $class
        }
        sub x { shift->{x} }
        sub y { shift->{y} }
    }

And even though object creation in Perl 6 is really optimized for using
named parameters, you **can** use positional parameters if you want to.
But you will have to create your own "`new`" method then:

    # Perl 6
    class Point {
        has $.x;
        has $.y;
        method new( $x, $y ) {
            self.bless( x => $x, y => $y )
        }
    }

Note that this looks very similar to the Perl 5 case.  But there are subtle
differences here.  In Perl 6, positional arguments are obligatory (unless
declared to be optional).  Making them optional with a default value, works
pretty much the same as with attribute declaration.  As well as indicating
a type: you specify those in the signature of the "`new`" method:

    # Perl 6
    ...
    method new( Int $x = 0, Int $y = 0 ) {
        self.bless( x => $x, y => $y )
    }

[DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) (Don't Repeat
Yourself) is a principle that you should always apply.  An example of making
it easier to DRY in Perl 6, is the syntactic sugar that exists for "`x => $x`"
(a [Pair](https://docs.perl6.org/type/Pair) in which the key has the same name
as the variable for the value).  In Perl 6 this can be expressed as "`:$x`".
So that would make the above "`new`" method look like:

    $ Perl 6
    ...
    method new( Int $x = 0, Int $y = 0 ) { self.bless( :$x, :$y ) }

After this, creating a `Point` object is again remarkably similar between
Perl 5 and Perl 6:

    # Perl 5
    my $point = Point->new( 42, 666 );

    # Perl 6
    my $point = Point.new( 42, 666 );

Summary
=======
Creating classes in Perl 6 is mostly declarative, whereas object creation in
standard Perl 5 is mostly procedural.  The way how classes are defined in
Perl 6 is very similar in semantics to
[Moose](https://metacpan.org/pod/distribution/Moose/lib/Moose/Manual.pod).
This is because `Moose` historically was inspired by the design of the Perl 6
object creation model, and vice-versa.

Performance concerns about object creation have always been a focus for
attention in both Perl 5 and Perl 6.  Even though Perl 6 provides much more
functionality in object creation than Perl 5, benchmarks show that Perl 6
has recently become faster than Perl 5 at creating and accessing objects.
