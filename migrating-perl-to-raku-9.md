# Subroutine Signatures
In this blog post we will focus on (subroutine) signatures in Raku and how argument passing is different from Perl.

## Experimental signatures in Perl
If you're migrating Perl code to Raku, you're probably not using the experimental signature feature that became available in Perl 5.20 or any of the older CPAN modules like `signatures`, `Function::Parameters`, or any of the other Perl modules on CPAN with signature in their name.

Also, in my experience `prototypes` haven't been used very often in the Perl programs out in the world (e.g., the "DarkPAN"). 

For these reasons, Raku functionality will only be compared with the most common use of "classic" Perl argument passing.

# Argument passing in Perl
All arguments you pass to a Perl subroutine are flattened and put into the automatically defined `@_` array variable inside. That is basically all Perl does with passing arguments to subroutines. Nothing more, nothing less. There are, however, several idioms in Perl that take it from there.

The most common (I would say "standard") idiom in my experience is:
```
# Perl
sub do_something {
    my ($foo, $bar) = @_;
    # actually do something with $foo and $bar
}
```
This idiom performs a list assignment (copy) to two (new) lexical variables. This way of accessing the arguments to a subroutine is also supported in Raku, but it's intended just as a way to make migrations easier.

If you expect a fixed number of arguments followed by a variable number of arguments, the following idiom is typically used in Perl:
```
# Perl
sub do_something {
    my $foo = shift;
    my $bar = shift;
    for (@_) {
        # do something for each element in @_
    }
}
```
This idiom depends on the magic behaviour of `shift` in Perl, which shifts from `@_` in this context. If the subroutine is intended to be called as a method, something like this is usually seen in Perl:
```
# Perl
sub do_something {
    my $self = shift;
    # do something with $self
}
```
as the first argument passed is the "invocant" in Perl.

By the way, this idiom can also be written in the first idiom:
```
# Perl
sub do_something {
    my ($self, @rest) = @_;
    for (@rest) {
        # do something for each element in @rest
    }
}
```
But that would be less efficient, as it would involve copying a potentially long list of values.

The third idiom revolves on directly accessing the `@_` array.
```
# Perl
sub sum_two {
    return $_[0] + $_[1];  # sum of the two parameters
}
```
This idiom is typically used for small, one-line subroutines, as it is one of the most efficient ways of handling arguments because no copying takes place.

This idiom is also used if you want to change any variable that is passed as a parameter. Since the elements in `@_` are aliases to any variables specified (in Raku you would say: "are bound to the variables"), it is possible to change their contents:
```
# Perl
sub make42 {
    $_[0] = 42;
}
my $a = 666;
make42($a);
say $a;      # 42
```
## Named arguments in Perl
Named arguments (as such) do *not* exist in Perl. But there is an often-used idiom that effectively mimics named arguments:
```
# Perl
sub frobnicate {
    my %named = @_;
    if (exists %named{bar}) {
        # do stuff if named variable "bar" exists
    }
}
```
This initialises the hash `%named` by alternately taking a key and a value from the `@_` array.

If you then call that subroutine with arguments using the fat-comma syntax:
```
# Perl
frobnicate( bar => 42 );
```
it will pass two values, "foo" and 42, which will be placed into the `%named` hash as the value 42 associated with key "foo".

But the same thing would have happened if you had specified:
```
# Perl
frobnicate( "bar", 42 );
```
The `=>` is syntactic sugar for automatically quoting the left side of it. Otherwise, it functions just like a comma (hence the name "fat comma").

If a subroutine is called as a method with named arguments, this idiom is combined with the standard idiom:
```
# Perl
sub do_something {
    my ($self, %named) = @_;
    # do something with $self and %named
}
```
alternatively:
```
# Perl
sub do_something {
    my $self  = shift;
    my %named = @_;
    # do something with $self and %named
}
```

## Argument passing in Raku
In their simplest form, subroutine signatures in Raku are very much like the "standard" idiom of Perl. But instead of being part of the code, they are part of the definition of the subroutine, and you don't need to do the assignment:
```
# Raku
sub do-something ($foo, $bar) {
    # actually do something with $foo and $bar
}
```
versus:
```
# Perl
sub do_something {
    my ($foo, $bar) = @_;
    # actually do something with $foo and $bar
}
```
In Raku, the `($foo, $bar)` part is called the "signature" of the subroutine.

Since Raku has an actual [`method`](https://docs.raku.org/type/Method) keyword, it is not necessary to take the invocant into account, as that is automatically available with the [`self`](https://docs.raku.org/language/objects#self) term:
```
# Raku
class Foo {
    method do-something-else ($foo, $bar) {
        # do something else with self, $foo and $bar
    }
}
```
Such parameters are called "positional" parameters in Raku. Unless indicated otherwise, positional parameters **must** be specified when calling a subroutine or a method.

If you need the aliasing behaviour of using `$_[0]` directly from Perl, you can mark the parameter as writable by specifying the `is rw` trait:
```
# Raku
sub make42 ($foo is rw) {
    $foo = 42;
}
my $a = 666;
make42($a);
say $a;      # 42
```
When you pass an array as an argument to a subroutine, it does **not** get flattened in Raku. You only need to accept an array as an array in the signature:
```
# Raku
sub handle-array (@a) {
    # do something with @a
}
my @foo = "a" .. "z";
handle-array(@foo);
```
You can pass any number of arrays:
```
# Raku
sub handle-two-arrays (@a, @b) {
    # do something with @a and @b
}
my @bar = 1..26;
handle-two-arrays(@foo, @bar);
```
If you want the ("variadic") flattening semantics of Perl, you can indicate this with a so-called "slurpy array" by prefixing the array with an asterisk in the signature:
```
# Raku
sub slurp-an-array (*@values) {
    # do something with @values
}
slurp-an-array("foo", 42, “baz");
```
A slurpy array can occur only as the last positional parameter in a signature.

If you prefer to use the Perl way of specifying parameters in Raku, you can do this by specifying a slurpy array *@_ in the signature:
```
# Raku
sub do-like-5 (*@_) {
    my ($foo, $bar) = @_;
}
```
Or even be more like Perl by using the auto-signature feature of Raku:
```
# Raku
sub also-like-5 {
    my ($foo, $bar) = @_;
}
```
The absence of a signature specification **and** the presence of `@_` in the body of the subroutine, will automatially create the `(*@_)` signature for you.  This feature makes direct migration of Perl code to Raku a lot simpler, but at the expense of more overhead because of the additional assignments.

## Named arguments in Raku
On the calling side, named arguments in Raku can be expressed very similarly to how they are expressed in Perl:
```
# Perl and Raku
frobnicate( bar => 42 );
```
However, on the definition side of the subroutine, things are very different:
```
# Raku
sub frobnicate (:$bar) {
    # do something with $bar
}
```
The difference between an ordinary (positional) parameter and a named parameter is the colon (`:`), which precedes the "sigil" and the variable name in the definition:
```
# Raku
$foo      # positional parameter, receives in $foo
:$bar     # named parameter "bar", receives in $bar
```
Unless otherwise specified, named parameters are *optional*. If a named argument is not specified, the associated variable will contain the default value, which usually is the type object `Any`.

If you want to catch any (other) named arguments, you can use a so-called "slurpy hash".  Just like the slurpy array, it is indicated with an asterisk before a hash:
```
# Raku
sub slurp-nameds (*%nameds) {
    say "Received: " ~ join ", ", sort keys %nameds;
}
slurp-nameds(foo => 42, bar => 666);# Received: bar, foo
```
As with the slurpy array, there can be only *one* slurpy hash in a signature, and it must be specified *after* any other named parameters.

Often you want to pass a named argument to a subroutine from a variable with the same name.  In Perl this looks like:
```
# Perl
do_something(bar => $bar);
```
In Raku, you can specify this in the same way:
```
# Raku
do-something(bar => $bar);
```
But you can also use a shortcut:
```
# Raku
do-something(:$bar);
```
Note that you do not need to type "bar" twice in Raku.  This means less typing – and less chance of typos.

## Default values in Raku
Perl has the following idiom for making parameters optional with a default value:
```
# Perl
sub dosomething_with_defaults {
    my $foo = @_ ? shift : 42;
    my $bar = @_ ? shift : 666;
    # actually do something with $foo and $bar
}
```
It basically checks whether there are any elements left in `@_`, and if there are, removes the first element and uses that as the value.  If there are no elements left, then use the alternate value.

In Raku, you can specify default values as part of the signature by specifying an equal sign and an expression:
```
# Raku
sub dosomething-with-defaults ($foo = 42, :$bar = 666) {
    # actually do something with $foo and $bar
}
```
Positional parameters become optional if a default value is specified for them.  Or can be made optional by postfixing a question mark (`?`).  Which is essentially the same as `= Any`.

Named parameters stay optional regardless of any default value.  But they can be made mandatory by postfixing the exclamation mark (`!`):
```
# Raku
sub dosomething-with-required-named (:$bar!) {
    # actually do something with $bar
}
```

# Summary
Raku has a way of describing how arguments to a subroutine should be captured into parameters of that subroutine. Positional parameters are indicated by their name and the appropriate sigil (e.g., `$foo`). Named parameters are prefixed with a colon (e.g. `:$bar`). Parameters can be marked as `is rw` to allow changing variables in the caller's scope.

Positional arguments can be flattened in a "slurpy" array, which is prefixed by an asterisk (e.g., `*@values`). Unexpected named arguments can be collected using a "slurpy" hash, which is also prefixed with an asterisk (e.g., `*%nameds`).

Default values can be specified inside the signature by adding an expression after an equal sign (e.g., `$foo = 42`), which also makes that parameter optional.

Signatures in Raku have many other interesting features, aside from the ones summarised here; if you want to know more about them, check out the Raku [`signature`](https://docs.raku.org/language/signatures) documentation.
