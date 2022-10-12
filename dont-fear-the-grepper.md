# Don't fear the grepper!

This blog post provides an introduction to the Raku Programmming Language and its [`grep` functionality](https://docs.raku.org/routine/grep#(List)_routine_grep).  It does not require any specific knowledge about the Raku Programming Language, although being familiar with basic `grep` (as a unix utility) functionality, is recommended.

The `grep` functionality comes in two flavours in Raku: a procedural (`sub`) version, and an object oriented (`ethod` version.  Since everything in Raku is an object (or can be thought of as one), and I personally mostly prefer the object oriented way, I will be discussing only the `method` way of using `grep` and friends.

## A simple example

Suppose we have an array with the numbers 1 through 10.  And we would like to put the even numbers in another array.  We can do this with the `grep` method, which takes a piece of code as an argument to determine whether a value should be included or not.
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
- values are returned with `return` from a subroutine
- arrays have a `@` prefix (usually referred to as a `sigil`)
- calling a method is done by placing a period between object and method name
- a subroutine can be referred to by prefixing its name with `&`


