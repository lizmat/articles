# Don't fear the grepper!

This blog post provides an introduction to the Raku Programmming Language and its [`grep` functionality](https://docs.raku.org/routine/grep#(List)\_routine_grep).  It does not require any specific knowledge about the Raku Programming Language, although being familiar with basic `grep` (as a unix utility) functionality, is recommended.

The `grep` functionality comes in two flavours in Raku: a procedural (`sub`) version, and an object oriented (`method`) version.  Since everything in Raku is an object (or can be thought of as one), and I personally mostly prefer the object oriented way, I will be discussing only the `method` way of using `grep` and friends.

## A simple example

Let's start with a simple question: Show the even numbers between 1 and 10.  That should be easy enough.

We create an array with the numbers 1 through 10.  And we put the even numbers in another array, which we can do this with the `grep` method.  The `grep` method takes a piece of code as the argument: this will then be repeatedly called with a value to determine whether or not that value should be included.
```raku
sub is-even($number) {    # returns True if given number is even
    return $number %% 2;  # %% is the "evenly divisible" operator
}
my @whole = 1..10;                 # fill up the array
my @even = @whole.grep(&is-even);  # fill array with even numbers
say @even;  # [2 4 6 8 10];        # show the result
```
Let's dissect this piece of code: the first 3 lines contain the definition of a subroutine called "is-even".  It expects a single argument `$number`, and returns either `False` or `True` depending on whether the given value is even or not.

The fourth line defines (`my`) and initializes (`=`) an array called `@whole` with the numbers `1` through `10`.

The fifth line defines (`my`) and initializes (`=`) an array called `@even` with the result of calling the method `grep` (with the subroutine `even` as the argument) on the `@whole` array.

The sixth line shows the contents of the `@even` array.

So what should we learn about Raku from this example?

- subroutines are made with `sub`
- variables with a single value have a `$` prefix (usually referred to as a `sigil`)
- values are returned with `return` from a subroutine
- arrays have a `@` prefix (`sigil`)
- calling a method is done by placing a period `.` between object and method name
- a subroutine can be referred to by prefixing its name with `&` (sigil)

## First simplification

However, that seems like a lot of code to obtain the simple answer to the question: "Show the even numbers between 1 and 10 inclusive".  So let's start by simplifying this code:
```raku
sub is-even($number) { $number %% 2 }  # return not needed
say (1..10).grep(&is-even);            # also works on ranges
```
Note that we removed the `return` from the subroutine declaration.  By default, the last expression evaluated in any block of code (recognizable by the curly braces `{` `}`) will be returned automatically.

Also note that we removed the use of arrays altogether: `(1..10)` in Raku is a [`Range` object](https://docs.raku.org/type/Range) that can also handle `grep` method.

And the `say` subroutine will take any expression as its argument, so we're good here as well.

## Second simplification

However, we can even further simplify this code.  The subroutine "is-even" doesn't actually need a name, it just needs to be a piece of code.  A further simplification would thus be:
```raku
-> $number { $number %% 2 }
```
In Raku this is referred to as a ["pointy block"](https://docs.raku.org/language/functions#index-entry-pointy_blocks).  You could think of this as subroutine without name, but there's a subtle difference with a `sub`: you **can** `return` from a `sub`, but you can **not** return from a pointy block.  If you execute a `return` in a pointy block, it will return out of the first outer subroutine.

## Third simplification

In programming one often talks about [**DRY** as in Don't Repeat Yourself](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself).  Note that in the second simmplification, we removed the repeated mentions of `@whole` and `@even`.  In this third simplification, we will remve the repeated mention of `$number`:
```raku
{ $^number %% 2 }
```
This uses the [placeholder variable](https://docs.raku.org/language/variables#The_^_twigil) feature of Raku.  It is basically a shorthand for the code of the second simplification.

But why would we need to have a name for a variable in such a simple expression anyway?  Couldn't we do that in an even shorter way?  Well, in Raku you can, thanks to something called ["whatever currying"](https://docs.raku.org/type/Whatever#index-entry-Whatever-currying):
```raku
* %% 2
```
What?  That's it?  Yup.  Expressive, isn't it?  So the actual code to show the even numbers between 1 and 10 inclusive, becomes:
```raku
say (1..10).grep(* %% 2);
```
