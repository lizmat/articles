# Don't fear the grepper! (6)

This is part 6 of the ["Don't fear the grepper!"](https://dev.to/lizmat/dont-fear-the-grepper-1-1k3e) series.

## Storing results

In all of the previous instalments of this series of blog posts, the result of a `.grep` or `.map` operation was always immediately output with `say`.  The `say` subroutine is supplied by the Raku core: it calls the `.gist` method on the given object(s), which is expected to give you a... *gist* (as in the general meaning of a text).

For example:
```
say (1..5).map(* + 2);
```
outputs:
```
(3 4 5 6 7)
```
Note that the gist of the result of the `.map` is shown with parentheses to give you an idea of the listiness of the result.

> All objects in Raku have a `.gist` method, inherited from `Any`.  If you don't like the gist it produces for your classes, you will need to provide your own `method gist`.

You can also store the results of a `.grep` or `.map` in an array:
```
my @result = (1..5).map(* + 2);
say @result;
```
which outputs:
```
[3 4 5 6 7]
```
Note that this shows the result using square brackets.  That's because the `.gist` method on `Array` objects uses square brackets.  Which is otherwise all pretty straightforward.

You can even see the values as they're being calculated if you want to:
```
my @result = (1..5).map({ say "calculating $_"; $_ + 2});
say "stored";
say @result;
```
which outputs:
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
In this case, all possible values were calculated and actually stored in the array `@result` even though you were only interested in the first value (the first element in the array).  Which may be costly if there are a lot of values to calculate.

## Being efficient

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

What is more important to note, is that "stored" appears **before** you can see the values being calculated.  It almost looks like the `.map` is not getting executed, until we actually need to make a *gist* of it in order to be able to `say` it.  And you'd be right!

This is one of the properties of the Raku Programming Language: it tries to do as little as possible, and only do the stuff that's needed when its needed.

But you may ask, why did it fill the array completely in the example with `@result`?  In short, That's because it was decided that when an array is assigned to, it will keep filling until the right hand side of the assignment has been exhausted.

> Actually, it's a little more general than that, but this should be enough explanation for now

So you can think of:
```
my @result = (1..5).map({ say "calculating $_"; $_ + 2});
```
as:
```
my @result;
for (1..5).map({ say "calculating $_"; $_ + 2}) {
    @result.push($_);
}
```
It was decided that any other sort of (default) behaviour would have been too confusing, coming from other programming languages.

## It's an object

Remember:

> Everything in Raku is an object, or can appear to be one

The result of a `.map` is **also** an object.  And you can you use the [`.WHAT` method](https://docs.raku.org/language/mop#index-entry-syntax_WHAT-WHAT) to interrogate what kind of object something is.  Some examples:
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

Note that calling `.WHAT` on the `Seq` object, was completely silent otherwise.  That's because it didn't execute anything.  Because it didn't need to.  Because it just interrogated meta-information about the `Seq` object.

## Indexing a Seq

So what will happen if you want to see the first element only of a `Seq` object stored in a scalar variable?  You can use the same indexing as we did on the `@result` array, because `Seq` objects understand that:
```
my $seq = (1..5).map({ say "calculating $_"; $_ + 2});
say "stored";
say $seq[0];
```
which outputs:
```
stored
calculating 1
3
```
Wow.  It only calculated a single value!  Yes, here Raku could be as efficient as possible, because you only needed the first element.  But what if you also want the third element?
```
my $seq = (1..5).map({ say "calculating $_"; $_ + 2});
say $seq[0];
say $seq[2];
```
outputs:
```
calculating 1
3
calculating 2
calculating 3
5
```
As you can see, it doesn't re-calculate the first element again.  So yes, it looks like it is cached somewhere.  And you'd be right again.  As soon as you use indexing on a `Seq`, it will create a hidden array that will be used to cache previously calculated values.  Technically, that's because the `Seq` class performs the [`PositionalBindFailover`](https://docs.raku.org/type/PositionalBindFailover) role.

## From here to infinity

It's this efficiency in Raku that allows you to actually specify [`Inf`](https://docs.raku.org/type/Num#index-entry-Inf_(definition)) or [`Whatever`](https://docs.raku.org/type/Whatever) as the end-point of the range in our example:
```
my $seq = (1..*).map({ say "calculating $_"; $_ + 2});
say $seq[0];
```
which outputs:
```
calculating 1
3
```
without being busy calculating all values until the end of time or memory.

## Grep the mapper

One nice feature of `Seq`, is that you can call `.grep` (or `map` for that matter, or vice-versa) on it as well.  Going all the way back to the initial example of filtering on even numbers.  In this example, we first create a `Seq` object with `.map`, then create a new `Seq` object using `.grep` on that.  And then show the first three elements (`[0..2]`):
```
my $seq = (1..*).map({ say "map $_"; $_ + 2});
$seq = $seq.grep({ 
    say $_ %% 2 ?? "accept $_" !! "deny $_"; 
    $_ %% 2
});
say $seq[0..2];
```
which outputs:
```
map 1
deny 3
map 2
accept 4
map 3
deny 5
map 4
accept 6
map 5
deny 7
map 6
accept 8
(4 6 8)
```
This shows that all values are produced one-by-one through the chain as efficiently as possible.

Now how that all works under the hood, is going to be the subject of a slightly more advanced series of blog posts, tentatively titled "A gaze of iterators".

## Conclusion

This concludes the sixth and final part of the series.  It shows that the Raku Programming Language has a `Seq` object that is responsible for producing values.  And that there is a [`.WHAT` method](https://docs.raku.org/syntax/WHAT) that gives you the type object of an instance.  While sneakily introducing the [ternary `??` `!!` operator](https://docs.raku.org/routine/%3F%3F%20!!).

Questions and comments are always welcome.  You can also drop into the [#raku-beginner](https://web.libera.chat/?channel=#raku-beginner) channel on Libera.chat, or on Discord if you'd like to have more immediate feedback.

I hope you liked it! Thanks again for reading all the way to the end.
