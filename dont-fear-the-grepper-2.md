# Don't fear the grepper! (Part 2)

This blog post is a follow-up on [Don't fear the grepper! (Part 1)](https://dev.to/lizmat/dont-fear-the-grepper-1-1k3e), recommended to read first if you haven't already.

## A confession

When I said:

> The grep method takes a piece of code as the argument: this will then be repeatedly called with a value to determine whether or not that value should be included.

I was in fact not telling the entire truth.  The `grep` subroutine / method will take just about anything as the argument to filter on, in a process called "smart-matching".  Smart-matching basically means a form of comparison of two objects which somehow decides whether there is a match or not.  The most visible form of that is the [`~~` infix operator](https://docs.raku.org/language/operators#index-entry-smartmatch_operator), but that is basically just syntactic sugar for an underlying mechanism.

Other programming languages have also tried an implementation of smart-match.  From that, the Raku Programming Language has learned that:

1. any smart-matching must be **asymmetrical**.
2. any smart-matching must be **configurable** for each combination of objects.

## Why a ~~ b is not the same as b ~~ a

In hindsight, it seems odd that anybody thought that having smart-match be symmetrical, would be a good idea.  Let's take the example of a duck: A duck is a bird, but not all birds are ducks!  So, in pseudo code, `Duck ~~ Bird` should be True, and `Bird ~~ Duck` should be False.  Any symmetry there would be...   weird.

## Why it must be configurable

The Raku Programming Language comes with many classes of objects built in.  But any serious development of code in Raku, will see new classes of objects.  Either by yourself, or as part of a module in the [Raku ecosystem](https://raku.land).  Of course, Raku can provide some sensible defaults in smart-matching, but any developer should be able to specify the behaviour of smart-matching between any custom objects and any other (core or non-core) objects.

## The .ACCEPTS method
Smart-matching in Raku is implemented by the [`.ACCEPTS` method](https://docs.raku.org/routine/ACCEPTS).  All core classes have one implemented, either directly or through inheritance.  And you can program an `.ACCEPTS` method in your own classes: that's configurability for you!

So what happens if you write:
```
$a ~~ $b
```
That construct is basically syntactic sugar for:
```
$b.ACCEPTS($a)
```
The left-hand side of the `~~` operator becomes the argument a call to the `.ACCEPTS` method on the right-hand side of the `~~` operator.

Going back to the context of the examples of `grep` in the first blog post:
```
say (1..10).grep(* %% 2); # (2 4 6 8 10)
```

What happens under the hood, is that whatever you've given as argument to `grep`, will have its `.ACCEPTS` method called repeatedly for all values given.  And since we've given a piece of code as the argument, it will call the `.ACCEPTS` method on the underlying `Callable` object.  And `.ACCEPTS` method on a `Callable` object will execute the code with the given argument, and return whatever the result was of executing the code.  Which is then interpreted by the `grep` logic to include the value (trueish) or not (not-trueish).

Being very verbose, you could think of the above code doing:
```
my @result;
my $grepper = * %% 2;
for 1..10 -> $number {
    if $grepper.ACCEPTS($number) {
        @result.push($number);
    }
}
say @result.List; # (2 4 6 8 10)
```
Although the actual implementation takes some shortcuts, and is lazy.  But that's stuff for another series of blog posts.

## Smart-matching against types and roles
You can us smart-matching with type objects to filter out the objects of a certain type, or which do a certain role:
```
my @s = "a", "b", 42, "c", "d", 666, 137, π;
say @s.grep(Int);      # (42 666 137)
say @s.grep(Str);      # (a b c d)
say @s.grep(Numeric);  # (42 666 137 3.141592653589793)
```
The first `grep` smart-matches against the [`Int`](https://docs.raku.org/type/Int) type object, which is the type for integer values.  By the way, in Raku, this means integers of unlimited size!

The second `grep` smart-matches against the [`Str`](https://docs.raku.org/type/Str) type object, which is the type for string values.

> For the more Unicode savvy: in Raku, these are *always* normalized to NFG (Normalization Form Grapheme).  You can think of that as a [Normalization Form C](https://unicode.org/reports/tr15/#Norm_Forms) on steroids, which tries to combine characters as observed by a human, into a *single* real (or virtual) codepoint.

The third `grep` smart-matches against the [`Numeric`](https://docs.raku.org/type/Numeric) role.  This also includes the value for `π`, which is not an integer, but is definitely numeric.  Note that you can also spell `π` as `pi` in Raku.

## Smart-matching against values
Of course, you can also smart-match against values.  Let's go back to the original `grep` example, but check for a single value:
```
say (1..10).grep(2);  # (2)
```
That works, because values that are smart-matched against themselves, (generally) return C<True>.  But how would you express if you want to `grep` against more than one value?
