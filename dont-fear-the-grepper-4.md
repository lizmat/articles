# Don't fear the grepper! (4)

This blog post contains the fourth instalment of the Don't fear the grepper! series ([Part 1](https://dev.to/lizmat/dont-fear-the-grepper-1-1k3e), [Part 2](https://dev.to/lizmat/dont-fear-the-grepper-2-4ki5), [Part 3](https://dev.to/lizmat/dont-fear-the-grepper-3-hfp)) are all recommended reading, if you haven't read them already.

## Expanding the notion of grep

The `grep` method allows one to filter a list of values: either a value gets through, or it does not.  In this way, the functionality of `grep` is rather limited.  What if you would not only like to filter out unwanted values, but also would like to *adapt* an acceptable value on the fly?  Or turn a single value into multiple values?  With the `map` method, you can!

The [`map`](https://docs.raku.org/routine/map) method provides a superset of the functionality of `grep`, but you can also use it as `grep` provided you're using a piece of code to do the filtering (instead of smart-matching).  In many ways, understanding `map` well, will make understanding a lot of Raku a lot easier!

## Using map as a grep

Let's go back to the original example of `grep` in this series, this time using the topic variable in a (pointy) block:
```
say (1..10).grep({ $_ %% 2 }); # (2 4 6 8 10)
```
You could write this using `map` as:
```
say (1..10).map({
    if $_ %% 2 {
        $_
    }
    else {
        Empty
    }
}); # (2 4 6 8 10)
```
As we've seen before with blocks, the last value evaluated will be returned by the block.  So if the condition `$_ %% 2` is True, it will return `$_`, else it will return `Empty`.  Now, what is `Empty`, you might ask?

## Slipping away

The Raku Programming Language has a number of special values that have special meanings.  One of them is `Empty`, [`Slip`](https://docs.raku.org/type/Slip) without any elements.  A `Slip` is a special type of list that automatically flattens in any outer list-like structure.

So if the condition `$_ %% 2` is **not** true, an empty Slip will be put into the result of the `map`, and thus **not** produce a value.  So effectively removing the current value of `$_` from the list.

But you are not limited to slipping an empty list!  You can use the [`slip`](https://docs.raku.org/routine/slip) subroutine to create a Slip with any values you give it.  For example:
```
say (1..10).map({
    if $_ %% 2 {
        slip($_ - .5, $_ + .5)
    }
    else {
        Empty
    }
}); # (1.5 2.5 3.5 4.5 5.5 6.5 7.5 8.5 9.5 10.5)
```
In this example the even numbers are replaced by two values, one .5 less and one .5 more than the topic.

## Consequences of not slipping

So, what would happen if you do **not** slip these values, but just produce the two values?
```
say (1..10).map({
    if $_ %% 2 {
        $_ - .5, $_ + .5
    }
    else {
        Empty
    }
}); # ((1.5 2.5) (3.5 4.5) (5.5 6.5) (7.5 8.5) (9.5 10.5))
```
You would get a 5-element list of 2-element lists.  Instead of a single 10-element list.  In other words, the produced lists do **not** get flattened.

## Automatic emptying

Using an `if` / `else` structure is still pretty verbose though, and yes this can be expressed in a shorter way:
```
say (1..10).map({
    if $_ %% 2 {
        slip($_ - .5, $_ + .5)
    }
}); # (1.5 2.5 3.5 4.5 5.5 6.5 7.5 8.5 9.5 10.5)
```
What?  You just removed the `else`?  Yup!  The reason this works, is that the value of a failed `if` (or `elsif` for that matter), is `Empty`.  So you don't have to specify the `else` clause explicitely!

Still, it feels this could still be shorter.  And you'd be right!  Since there is no `else` in the code anymore, you can use the "statement modifier" version of `if` (sometimes also referred to as "postfix if"): 
```
say (1..10).map({
    slip($_ - .5, $_ + .5) if $_ %% 2
}); # (1.5 2.5 3.5 4.5 5.5 6.5 7.5 8.5 9.5 10.5)
```
That reduces on curly braces, but is also known to not improve readability for *some* people.  To yours truly, it feels very natural language-like.  Like everything in life, YMMV!

## Helicopter view

Whenever you're coding something to reach a certain goal, you should always think about the way you got to a solution.  And think about whether there couldn't be an easier way to reach the same result.

This is not just about efficiency of code execution, but also about whether your code shows the intent clearly or not.  And clear intent will make it easier for you, or anybody else, now or in the future, to [grok](https://en.wikipedia.org/wiki/Grok) the source code.

In this case, producing the values 1.5 through 10.5 with an interval of 1, can be done **much** easier, because with `map` we can also *change* values unconditionally!
```
say (1..10).map({ $_ + .5 });
# (1.5 2.5 3.5 4.5 5.5 6.5 7.5 8.5 9.5 10.5)
```
Much simpler, isn't it?  It is even so simple that we can use ["whatever-currying"](https://docs.raku.org/type/Whatever#index-entry-Whatever-currying)!
```
say (1..10).map(* + .5);
# (1.5 2.5 3.5 4.5 5.5 6.5 7.5 8.5 9.5 10.5)
```
And there you have a hopefully easy to grok use of the `map` method in a nutshell!

## Conclusion
This concludes the fourth part of the series, this time introducing the `map` method.  And also introducing the concept of `Empty`, and `Slip`s in general.  And also showing that you can have a statement modifier version of `if` if you don't need an `else` or an `elsif`.!

Questions and comments are always welcome.  You can also drop into the [#raku-beginner](https://web.libera.chat/?channel=#raku-beginner) channel on Libera.chat, or on Discord if you'd like to have more immediate feedback.

I hope you liked it! Thanks again for reading all the way to the end.
