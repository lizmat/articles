# A gaze of iterators! (Part 1)

This blog post provides an introduction to iterators in the [Raku Programming Language](https://raku.org).

It requires some basic understanding of Raku code.  One could consider the [Don't fear the grepper! series](https://dev.to/lizmat/dont-fear-the-grepper-1-1k3e) as a prerequisite for this series of blog posts.

## Iterator Central

Iterators are at the basis of every type of iteration in the Raku Programming Language, except for `loop` (which uses a counter or iterates indefinitely), `while` and `until` (which iterate while a condition is `True` / `False`).

Iterators are *everywhere* in Raku: all values and classes support having the `iterator` method called on them.
```
say 42.iterator;  # Rakudo::Iterator::ReifiedListIterator.new
```
So why would that say what it does?  Well, as we've seen before, the `say` subroutine will call the `.gist` method on whatever it got.  And the default (inherited) `.gist` method on instances of classes shows the name of the class and how it could *possibly* be created.

Same for any other class, even one of your own:
```
class Foo { }
say Foo.new;  # Foo.new
```

## What can you use an iterator for?

Good question!  Actually, in general you wouldn't be using an iterator yourself in your code.  You would provide Raku with an iterator (usually implicitly), and let that do the work for you.  In general.

But to get the feel of what an iterator can do, we're going to tinker with iterators a bit in this series of blog posts.  Just to get a feel of what is going on under the hood, as it were.

## Looking on the inside

First of all, one would like to know which methods you can call on an iterator object.  Fortunately, the Raku Programming Language has many introspection capabilities.

One of them is the `.methods` method on the meta-object of the iterator.  Don't think too much about that at this point: just know that there's a special syntax for calling a method on the meta-object of an object: [`.^method-name`](https://docs.raku.org/language/operators#methodop_.^).

So let's see what the iterator on `42` can do:
```
.say for 42.iterator.^methods;
# new
# pull-one
# push-exactly
# push-all
...
```
`.new`?  There's nothing new about that?  Well, yes and no.  The fact that it is listed here, means that the class has its **own** (not inherited) `method new`.  Whether that is useful information, is up to the reader!

The next one is `.pull-one`.  Let's see what happens if we call that on the iterator object:
```
say 42.iterator.pull-one; # 42
```
I guess you could hardly call that surprising.  But what happens if you would call the `.pull-one` method for a second time?
```
my $iter = 42.iterator;
say $iter.pull-one; # 42
say $iter.pull-one; # IterationEnd
```
Hmmmm... what's this [`IterationEnd`](https://docs.raku.org/type/Iterator#index-entry-IterationEnd) you say?  It's a very special sentinel value that indicates that the iterator is **done** producing values.  And that you should not call any methods on the iterator anymore (as the results will be undefined).

Ok, so let's try this again, this time with a small list of values:
```
my $iter = <a b c>.iterator;
say $iter.pull-one; # a
say $iter.pull-one; # b
say $iter.pull-one; # c
say $iter.pull-one; # IterationEnd
```
Or, shorter:
```
my $iter = <a b c>.iterator;
say $iter.pull-one for ^4;
# a
# b
# c
# IterationEnd
```

## Pulling until it's done

Now that we know that the final value is `IterationEnd`, we should be able to write a loop checking for that value, right?  And end the loop on that?  Indeed we can!  But it requires some special care:
```
my $iter = <a b c>.iterator;
until ($_ := $iter.pull-one) =:= IterationEnd {
    .say;
}
# a
# b
# c
```
That's maybe a lot of colons all of a sudden!

The first colon is in [`:=`](https://docs.raku.org/routine/:=) which is the binding operator. It aliases the left side with the right side: in this case with the topic [`$_`](https://docs.raku.org/syntax/$_). It's to make sure that we're going to compare the actual value directly, rather than a value in a variable (as values in variables may actually appear differently to the outside world, if they want to).

The second one is the [`=:=`](https://docs.raku.org/routine/=:=), the identity operator.  It checks whether both sides refer to the same item in memory.  Whether they are *really* the **same** object.

Looking at this code, you might realize that this is actually a convoluted way to write:
```
for <a b c> {
    .say
}
```
And you'd be right: what you see above is more or less essentially what is happening under the hood.  Of course, reality is a bit more complicated: in this example we for instance didn't account for handling any loop phasers ([`FIRST`, `NEXT` and `LAST`](https://docs.raku.org/language/phasers#index-entry-Phasers__FIRST-FIRST)).  But the basic principle is the same!

And because every value and every class can take a call to the `iterator` method, you should understand now why this works:
```
for 42 {
    .say
}
# 42
```
Indeed, because you *can* call the `.iterator` method on any class or object!

## Conclusion

This concludes the first part of the introduction to iterators, and possibly to the Raku Programming Language.

It introduced the `iterator` and `^methods` methods, as well as the `pull-one` method and the special `IterationEnd` sentinel value for iterators.  And it casually introduced the `:=` binding operator and the `=:=` identity operator.

Questions and comments are always welcome.  You can also drop into the [#raku-beginner channel](https://web.libera.chat/?channel=#raku-beginner) on Libera.chat, or on Discord if you'd like to have more immediate feedback.

I hope you liked it!  Thank you for reading all the way to the end.
