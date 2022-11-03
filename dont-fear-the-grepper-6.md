# Don't fear the grepper! (6)

This blog post contains the sixth instalment of the Don't fear the grepper! series.

## Storing results

In all of the previous instalments of this series of blog posts, the result of a `.grep` or `.map` operation was always immediately output with `say`.  The `say` subroutine is supplied by the Raku core: it calls the `.gist` method on the given objects, which is expected to give you a... *gist* (as in the general meaning of a text).

> Note that all objects in Raku have a `.gist` method, inherited from `Any`.  If you don't like the gist it produces for your classes, you will need to provide your own `method gist`.

For example:
```
say (1..5).map(* + 2);
```
which would output:
```
(3 4 5 6 7)
```
Note that the gist of the result of the `.map` is shown with parentheses.

You can also store the results of a `.grep` or `.map` in an array:
```
my @result = (1..5).map(* + 2);
say @result;
```
which would output:
```
[3 4 5 6 7]
```
Note that this shows the result using square brackets.  That's because the `.gist` method on `Array` objects use square brackets.  Which is otherwise all pretty straightforward.

You can even see the values as they're being calculated if you want to:
```
my @result = (1..5).map({ say "calculating $_"; $_ + 2});
say "stored";
say @result;
```
which would output:
```
calculating 1
calculating 2
calculating 3
calculating 4
calculating 5
stored
[3 4 5 6 7]
```

And if you're not interested in the complete result, you could just ask for it to show the first element.  And as indexing in Raku is zero-based, that would be index *0*:
```
my @result = (1..5).map({ say "calculating $_"; $_ + 2});
say "stored";
say @result[0];
```
which would output:
```
calculating 1
calculating 2
calculating 3
calculating 4
calculating 5
stored
3
```
Note that all possible values were calculated and actually stored in the array `@result`, even though you were only interested in the first value (the first element in the array).  Which may be costly if there are a lot of values to calculate.

## Being lazy

But couldn't you store the result in a scalar variable, and get the same result?  Yes, you can, but the flow of execution and the result would be subtly different:
```
my $result = (1..5).map({ say "calculating $_"; $_ + 2});
say "stored";
say $result;
```
which would output:
```
stored
calculating 1
calculating 2
calculating 3
calculating 4
calculating 5
(3 4 5 6 7)
```
Note that we're back to showing the final result using parentheses.  That's really because we're in fact just "gisting" (as one would say as an experienced Rakoon) the result of the `.map` like the original example.

What is more important to note, is that "stored" appears **before** you can see the values being calculated.  It almost looks like the `.map` is not getting executed, until we actually need a *gist* of it in order to be able to `say` it.  And you'd be right!  Remember:

> Everything in Raku is an object, or can appear to be one

So, the result of a `.map` is **also** an object.  And you can you use the [`.WHAT` method](https://docs.raku.org/language/mop#index-entry-syntax_WHAT-WHAT) to interrogate what kind of object something is:
```
say 42.WHAT;     # (Int)
say "foo".WHAT;  # (Str)
say (1..5).WHAT; # (Range)
```
Note that `.WHAT` returns the [type object](https://docs.raku.org/language/classtut#index-entry-type_object) (aka the class) of an object.  And the `.gist` method for type objects, puts parentheses around the name as an indicator it is a type object.

So what type of object is returned by `.map`?
```
say (1..5).map({ .say; $_ + 2}).WHAT; # (Seq)
```
A [`Seq`](https://docs.raku.org/type/Seq) object. Aha!.

Note that calling `.WHAT` on the `Seq` object, was completely silent otherwise.  That's because it didn't execute anything.  Because it didn't need to.  Because it was lazy!

So what will happen if you want to see the first element only?
```
my $result = (1..5).map({ say "calculating $_"; $_ + 2});
say "stored";
say $result[0];
```
which would output:
```
stored
calculating 1
3
```



## Conclusion

This concludes the sixth and final part of the series.

Questions and comments are always welcome.  You can also drop into the [#raku-beginner](https://web.libera.chat/?channel=#raku-beginner) channel on Libera.chat, or on Discord if you'd like to have more immediate feedback.

I hope you liked it! Thanks again for reading all the way to the end.
