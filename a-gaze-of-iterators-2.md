# A gaze of iterators! (Part 2)

## Pushing the limits

So let's again look at the list of methods the iterator on `42` has:
```
.say for 42.iterator.^methods'
# new
# pull-one
# push-exactly
# push-all
# skip-one
# skip-at-least
# count-only
# sink-all
# bool-only
# push-until-lazy
# push-at-least
# skip-at-least-pull-one
# is-lazy
# is-deterministic
# BUILDALL
...
```
Hmmm... `.push-all` looks interesting.  Could it really be that simple?  Let's see!

```
my @array;
42.iterator.push-all(@array);
say @array;  # [42]
```
And how about a list of values?
```
my @array;
<a b c>.iterator.push-all(@array);
say @array;  # [a b c]
```
Looking at this, you could realize that the above is just a convoluted way to write:
```
my @array = <a b c>;
say @array;  # [a b c]
```
And you'd be right again!  And now you have a better idea of what goes on under the hood.  Well, at least conceptually, because the actual implementation is of course at liberty to take short-cuts to improve efficiency.

## Don't like that one

The next method on that list is `.skip-one`.  Looks like it's mostly like `.pull-one`, so let's check:
```
my $iter = <a b c>.iterator;
say $iter.skip-one; # 1
say $iter.pull-one; # b
say $iter.pull-one; # c
say $iter.skip-one; # 0
```
So it looks like `.skip-one` really skips a value.  But it also returns something?  Indeed, it returns either `1` to indicate a successful skip, and `0` for an unsuccessful skip.  Why not `True` and `False` you might ask?  Well, these are all methods that work under the hood as efficiently as possible, and turning a native integer `1` into a boolean `True` would just be extra and unnecessary work.

# What is it

Those two [`.is-lazy`](https://docs.raku.org/type/Iterator#method_is-lazy) and `is-deterministic` methods also look interesting:
```
say 42.iterator.is-lazy;           # False
say 42.iterator.is-deterministic;  # True
```
The `.is-lazy` method indicates whether the iterator is lazy or not.  In hindsight, the term "lazy" was probably a bad choice.  The most obvious thing about "lazy" iterators, is that you cannot calculate the number of elements it will produce.  So the term "countable" (with reversing the value) would probably have been better.
```
say (1..*).iterator.is-lazy;  # True
```
is an example of a "lazy" iterator, of which you can **not** count the number of elems.  If you try to do that with the `.elems` method, you will get an error:
```
say (1..*).elems;  # Cannot .elems a lazy list
```
If it wouldn't produce the error, it would hang because it would be producing values "ad infinitum" literally!  Before being able to tell you the number of elements.

The `is-deterministic` method indicates whether the iterator, given a certain source, will always produce the same values in the same order.  The Raku internals can optimize certain situations if it knows whether the produced values will always be the same.
```
say (1..10).iterator.is-deterministic;          # True
say (1..10).pick(*).iterator.is-deterministic;  # False
```
Note the [`.pick`](https://docs.raku.org/type/List#routine_pick) method will produce the given values in a random order, so clearly **not** [deterministic](https://en.wikipedia.org/wiki/Deterministic_algorithm)!
```
say (1..10).pick(*);  # (7 1 9 2 4 5 10 8 3 6)
say (1..10).pick(*);  # (4 6 8 9 2 10 7 5 1 3)
```

## That's weird

Yeah, what does `BUILDALL` do?  Actually, nothing that should concern you.  The ALLCAPS of the method really indicates that there is something special going on!  The `BUILDALL` method is a method that is automatically generated for every class, and it contains the default logic to initialize an object of that class.  There is no source code for it: the definition of a class determines how that method will be directly generated into executable bytecode.

## What are you sinking about?

Now, the other methods all have names that make sense, probably.  But there is one method that appears to be different: [`.sink-all`](https://docs.raku.org/routine/sink-all).  Let's see what happens if we call that:
```
say <a b c>.iterator.sink-all;  # IterationEnd
```
That's informational?  But what does it do?  Well, in this particular case, it will mark the iterator as completed.  Not very useful.

But there are other (very common) cases where calling `sink-all` *is* very useful.  Remember that a `for` loop is really just a `.map` of which the results are discarded?
```
my $seen = 0;
(1..10).map({++$seen}).iterator.sink-all;
say $seen;  # 10
```
The `sink-all` method is used internally for those iterators that are just executed for their side-effects.  So the above is just a very complicated way to write:
```
my $seen = 0;
++$seen for 1..10;
say $seen;  # 10
```
The term ["sink"](https://docs.raku.org/language/contexts#index-entry-sink_context) is the Raku equivalent for what other programming languages call "void context".  But more about that later in this series.

## Conclusion

This concludes the second part of the series, in which most of the other methods that you can call on an iterator have been explained.  Specifically `.skip-one`, `push-all`, `is-lazy`, `is-deterministic` and `sink-all`.  With a side-order of `.pick`.

Questions and comments are always welcome.  You can also drop into the [#raku-beginner channel](https://web.libera.chat/?channel=#raku-beginner) on Libera.chat, or on Discord if you'd like to have more immediate feedback.

I hope you liked it!  Thank you for reading all the way to the end.
