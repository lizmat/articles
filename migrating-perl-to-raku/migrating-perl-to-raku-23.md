# Optimisation Considerations
If you are an experienced Perl programmer, you have (perhaps inadvertently) learned a few tricks to make execution of your Perl program faster.  Some of these idioms work counter-productively in Raku.  This blog post deals with some of them and provides the alternative idioms in Raku.

## Hashes vs classes
Objects in Perl generally consist of blessed hashes.

As we’ve seen before, this implies that accessors need to be made for them.  Which means an extra level of indirection and overhead.  So many Perl programmers “know” that the object is basically a hash, and access keys in the blessed hash directly.

Consequently, many Perl programmers decide to forget about creating objects altogether and just use hashes.  Which is functionally ok if you’re just using the hash as a store for keys and associated values.  But in Raku it is better to actually use objects for that from a performance point of view.  Take this example where a hash is created with two keys / values:
```
# Raku
for ^1_000_000 { # do this a million times
    my %h = a => 42, b => 666;
}
say now - INIT now;  # 0.779620151
```
Now, if we use an object with two attributes, this is more than 2.5x as fast:
```
# Raku/
class A {
    has $.a;
    has $.b;
}
for ^1_000_000 {
    my $obj = A.new(a => 42, b => 666);
}
say now - INIT now;  # 0.278380754
```
But, you might argue, accessing the keys in the hash will be faster than calling an accessor to fetch the values?

Nope.  Using accessors in Raku is faster as well.  Compare this code:
```
# Raku
my %h = a => 42, b => 666;
for ^10_000_000 {  # do this ten million times
    my $a = %h<a>;
}
say now - INIT now;  # 1.040867083
```
To:
```
# Raku
class A {
    has $.a;
    has $.b;
}
my $obj = A.new(a => 42, b => 666);
for ^10_000_000 {
    my $a = $obj.a;
}
say now - INIT now;  # 0.372786587
```
Note that using accessors is also a lot faster.

So why is the accessor method faster in Raku?  Well, really because Raku is able to optimise the attributes in an object to a list, easily indexed by a number internally.  Whereas for a hash lookup, the string must be *always* hashed before it can be looked up.  And that takes a lot more work than just a lookup by index.

## Native variables
In standard Perl values are **always** "native", meaning that they're basically just a piece of memory that happens to be formatted as an integer, a floating point, or a string.

In Raku values are always objects by default, meaning that their values are always encapsulated in a Raku object.  This may be clear in the case of [`Rat`](https://docs.raku.org/type/Rat)s, but for integers and strings, this may be unexpected.

In the case of integer values, putting them into an object allows for transparent big integer support.  Yes, in Raku integer values are **not** limited to 64 bits.  But this comes at a price of course.

However, you *can* access [native values in Raku](https://docs.raku.org/language/nativetypes) as well: this is done by defining variables with a *native* type.  There are currently 4 kinds of them: `int`, `uint`, `num` and `str`.

Let's look at an integer increment example:
```
# Raku
my $a = 0;
for ^10_000_000 {
    $a++;
}
say now - INIT now;  # 0.395348719
```
Now with using native integers:
```
# Raku
my int $a = 0;
for ^10_000_000 {
    $a++;
}
say now - INIT now;  # 0.122372013
```
And for concatenating strings:
```
my $a = "";
for ^10_000_000 {
    $a ~= "x";
}
say now - INIT now;  # 0.997466791
```
When using native strings:
```
my str $a = "";
for ^10_000_000 {
    $a ~= "x";
}
say now - INIT now;  # 0.58488469
```
Note that using native values everywhere may actually (currently) be counterproductive.  This is because there are many situations (although fewer and fewer) where Raku has no option but to upgrade a native value to a Raku object before it can handle it.  This happens for instance when calling a method on a native value.

An example: converting a string to an integer.
```
# Raku
my $a = "42";
for ^1_000_000 {
    $a.Int
}
say now - INIT now;  # 0.300763207
```
And now with a native string:
```
# Raku
my str $a = "42";
for ^1_000_000 {
    $a.Int
}
say now - INIT now;  # 0.433718985
```
So in this case using a native `str` variable does **not** help in performance (at least at the moment of this writing).

Fortunately the runtime optimizer in Raku catches more and more of these cases and is then able to simplify these without needing to upgrade the native value every time it is being used.

## Summary
As with all benchmarks, these are just a snapshot in time.  They also depend on OS versions, Rakudo versions, hardware and load.  Optimisation work continues to be performed on Raku, which may change the outcome of these tests in the future.

So, always test yourself if you want to be sure that some optimisation is a valid approach, but only if you’re really interested in squeezing performance out of your Raku code.  Remember, premature optimisation is the root of all evil!
