# A gaze of iterators! (Part 1)

This blog post provides an introduction to iterators in the [Raku Programming Language](https://raku.org).  It requires some basic understanding of Raku code.  One could consider the [Don't fear the grepper! series](https://dev.to/lizmat/dont-fear-the-grepper-1-1k3e) as a prerequisite for this series of blog posts.

## Iterator Central

Iterators are at the basis of every type of iteration in the Raku Programming Language, except for `loop` (which iterates indefinitely), `while` and `until` (which iterate while a condition is `True` / `False`).

All values and classes support having an `iterator` method called on them.
```
say 42.iterator;  # Rakudo::Iterator::ReifiedListIterator.new
```
So why would that say what it does?  Well, as we've seen before, the `say` subroutine will call the `.gist` method on whatever it got.  And the default `.gist` method on instances of classes shows the name of the class and how it *possibly* be created.  Same for any other class, even one of your own:
```
class Foo { }
say Foo.new;  # Foo.new
```
In any case, because every value and every class can take a call to the `iterator` method, you can do things like:
```
for 42 {
    .say
}
# 42
```

## How do we know it's an iterator?

Calling the `iterator` method on something, gives us an object.  How do we know it's an iterator?  We could see if smartmatch can tell:
```
say 42 ~~ Iterator;           # False
say 42.iterator ~~ Iterator;  # True
```


## Conclusion

This concludes the first part of the introduction to iterators, and possibly to the Raku Programming Language.  Questions and comments are always welcome.  You can also drop into the [#raku-beginner channel](https://web.libera.chat/?channel=#raku-beginner) on Libera.chat, or on Discord if you'd like to have more immediate feedback.

I hope you liked it!  Thank you for reading all the way to the end.
