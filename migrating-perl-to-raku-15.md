# Classes (Part 2 of 2)
The second blog post investigating the differences in building classes in Perl and the Raku Programming Language.

## Providing mutators
In the class/object examples so far, an object's attributes have been immutable. They can't be changed by the usual means after the object has been created.

In Perl, there are various ways to create a mutator (a method on the object to change an attribute's value). The simplest way is to create a separate subroutine that will set the value in the object:
```
# Perl
…
sub set_x {
    my $object = shift;
    $object->{x} = shift;
}
```
which can be shortened to:
```
# Perl
…
sub set_x { $_[0]->{x} = $_[1] } # access elements in @_ directly
```
so you could use it as:
```
# Perl
my $point = Point->new( x => 42, y => 666 );
$point->set_x(137);
```
Some people prefer to use the same subroutine name for both accessing and mutating the attribute. Specifying a parameter then means the subroutine should be used as a mutator:
```
# Perl
…
sub x {
    my $object = shift;
    @_ ? $object->{x} = shift : $object->{x}
}
```
which can be shortened to:
```
# Perl
…
sub x { @_ > 1 ? $_[0]->{x} = $_[1] : $_[0]->{x} }
```
so you could use it as:
```
# Perl
my $point = Point->new( x => 42, y => 666 );
$point->x(137);
```
Here is a way this is used a lot in Perl, but it depends on the details with which objects are implemented. Since an object in Perl is usually just a hash reference with benefits, you can use the object as a hash reference and directly access keys in the underlying hash.

But this breaks the object's encapsulation and bypasses any additional checks that a mutator might do:
```
# Perl
my $point = Point->new( x => 42, y => 666 );
$point->{x} = 137;  # change x to 137 unconditionally: dirty but fast
```
An "official" way of creating accessors that can also be used as mutators uses "lvalue subroutines", but this isn't used often in Perl for various reasons.

It is however very close to how mutators work in Raku:
```
# Perl
…
sub x : lvalue { shift->{x} }  # make "x" an lvalue sub
```
So you could use it as:
```
# Perl
my $point = Point->new( x => 42, y => 666 );
$point->x = 137;  # just as if $point->x is a variable
```
In Raku, allowing an accessor to be used as a mutator is also done in a declarative way by using the `is rw` trait on the attribute declaration, just like with the `is required` trait:
```
# Raku
class Point {
    has Int $.x is rw = 0;
    has Int $.y is rw = 0;  # allowed to change, default is 0
}
```
This allows you to use it in Raku like this:
```
# Raku
my $point = Point.new( x => 42, y => 666 );
$point.x = 137;  # just as if $point.x is a variable
```
If you don't like the way mutators work in Raku, you can create your own mutators by adding a method for them. For example, the "set_x" case from Perl could look like this in Raku:
```
# Raku
class Point {
    has $.x;
    has $.y;
    method set-x($new) { $!x = $new }
    method set-y($new) { $!y = $new }
}
```
But wait: What's that exclamation point doing in `$!x` ???

The `!` indicates the *real* name of the attribute in the class; it gives direct access to the attribute in the object. Let's take a step back and see what the attribute's so-called ["twigil"](https://docs.raku.org/language/variables#Twigils) (i.e., the secondary sigil) means.

## The '!' twigil
A `!` in a declaration of an attribute like `$!x` designates that the attribute is *private*. This means you can't access that attribute from the outside unless the class' developer has provided a means to do so. This also means that it can not be initialised with a call to the `new` method.

A method for accessing the private attribute value can be very simple:
# Raku
class Point {
    has $!x;            # ! indicates a private attribute
    has $!y;
    method x() { $!x }  # return private attribute value
    method y() { $!y }
}
This is, in fact, pretty much what happens automatically if you declare the attribute with the `.` twigil:

## The '.' twigil
A period in a declaration of an attribute like `$.x` designates that the attribute is *public*. This means that an accessor method is created for it (much like the example above with the method for the private attribute). This also means the attribute can be initialised with a call to `new`.

If you otherwise use the attribute form `$.x`, you are **not** referring to the attribute, but rather to its accessor method. It is syntactic sugar for `self.x`. But the `$.x` form has the advantage that you can easily interpolate inside a string.

Furthermore, the accessor can be overridden by a subclass:
```
# Raku
class Answer {
    has $.x = 42;
    method message() {
        "The answer is $.x”;  # use accessor in message
    }
}
class Fake is Answer {    # subclassing is done with "is" trait
     method x() { 666 }   # override the accessor in Answer
}
say Answer.new.message;  # The answer is 42
say Fake.new.message;    # The answer is 666 (even though $!x is 42)
```

## Tweaking object creation
Sometimes you need to perform extra checks or tweaks to an object before it is ready for consumption. Without getting into the nitty-gritty of creating objects in Raku, you can usually do all the tweaking that you need by supplying a `TWEAK` method.

Suppose you also want to allow the value [137](https://en.wikipedia.org/wiki/137_(number)) to be considered as an alternative to 666:
```
# Raku
class Answer {
    has Int $.x = 42;
    submethod TWEAK() {
        $!x = 666 if $!x == 137;
    }
}
```
If a class has a `TWEAK` method, it will be called after all arguments have been processed and assigned to attributes, as appropriate (including assigning any default values and any processing of traits such as `is rw` and `is required`).  Inside the method, you can do whatever you want to the attributes in the object.

Note that the `TWEAK` method is best implemented as a so-called [`submethod`](https://docs.raku.org/language/objects#Submethods).  A `submethod` is a special type of method that can be executed only on the class itself and not on any subclass. In other words, this method has the visibility of a subroutine.
```

## Positional parameters
Finally, sometimes an interface to an object is so clear that you do not need named parameters at all. Instead, you want to use positional parameters.

In Perl, that would look something like this:
```
# Perl
package Point {
    sub new {
        my ($class, $x, $y) = @_;
        bless { x => $x, y => $y }, $class
    }
    sub x { shift->{x} }
    sub y { shift->{y} }
}
```
Even though object creation in Raku is optimised for using named parameters, you can use positional parameters if you want to. In this case, you'll have to create your own `new` method.

By the way, there is nothing special about the `new` method in Raku.  You can create your own, or you can create a method with another name to act as an object constructor:
```
# Raku
class Point {
    has $.x;
    has $.y;
    method new ($x, $y) {
        self.bless( x => $x, y => $y )
    }
}
```
This looks very similar to Perl, but there are subtle differences. In Raku, positional arguments are obligatory (unless they're declared to be optional).

Making them optional with a default value works pretty much the same as with attribute declaration, as does indicating a type: you specify those in the signature of the `new` method:
```
# Raku
…
method new (Int $x = 0, Int $y = 0) {
    self.bless( x => $x, y => $y )
}
```
The `bless` method provides the logic of object creation with given named parameters in Raku: its interface is the same as the default implementation of the `new` method. You can call it whenever you want to create an instantiated object of a class.

Don't repeat yourself (["DRY"](https://en.wikipedia.org/wiki/Don't_repeat_yourself)) is a principle you should always use. One example of making it easier to DRY in Raku is the syntactic sugar for `x => $x` (a `Pair` in which the key has the same name as the variable for the value).

In Raku, this can be expressed as `:$x`. That would make the above `new` method look like the following:
```
# Raku
…
method new (Int $x = 0, Int $y = 0) {
    self.bless( :$x, :$y )
}
```
After this, creating a `Point` object is again remarkably similar between Perl and Raku:
```
# Perl
my $point = Point->new( 42, 666 );
```
```
# Raku
my $point = Point.new( 42, 666 );
```

## Summary
Creating classes in Raku is mostly a declarative process, whereas object creation in standard Perl is mostly procedural. The way classes are defined in Raku is very similar in semantics to Moose. This is because Moose was inspired by the design of the Raku object creation model, and vice-versa.

Performance concerns about object creation have always been a focus in both Perl and Raku. Even though Raku provides more functionality in object creation than Perl, benchmarks show that Raku has become faster than Perl at creating and accessing objects.
