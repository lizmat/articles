# Don't fear the grepper! (5)

This blog post contains the fifth instalment of the Don't fear the grepper! series.

[Part 1](https://dev.to/lizmat/dont-fear-the-grepper-1-1k3e), [Part 2](https://dev.to/lizmat/dont-fear-the-grepper-2-4ki5), [Part 3](https://dev.to/lizmat/dont-fear-the-grepper-3-hfp), [Part 4](https://dev.to/lizmat/dont-fear-the-grepper-4-nki) are all recommended reading, if you haven't read them already.

## The next confession

Yes, another one.  In the previous blog post I said that you can remove (not accept) values in a `.map` by returning the `Empty` value from the block.
```
say (1..12).map({
    if $_ %% 2 {   # is it divisible by 2?
        $_         # yes, accept
    }
    else {         # not divisible by 2
        Empty      # don't accept
    }
}); # (2 4 6 8 10 12)
```

There's actually another way to not accept the value, and that is using the [`next` control flow statement](https://docs.raku.org/syntax/next).  With `next`, you're actually telling `.map` to stop executing any code inside the block immediately, and start the **next** iteration.  So how would that look in the above case?
```
say (1..12).map({
    next unless $_ %% 2;  # not divisible by 2, next!
    $_                    # accept
}); # (2 4 6 8 10 12)
```
Note that using `next` will interrupt the normal flow of the program.  When it executes, it will look up the call stack and instruct the first handler capable of handling `next` to immediately continue with the next iteration.  In this case, the `.map` has such a handler.

And you see I used [`unless`](https://docs.raku.org/syntax/unless), instead of [`if`](https://docs.raku.org/syntax/if) [`not`](https://docs.raku.org/routine/not) there.  I generally use `unless` only as a statement modifier, because using it with blocks generally doesn't improve readability, and therefor maintainability of any codebase.  But of course, I could also have written `next if not $_ %% 2`!

Other than the program flow interruption feature of `next`, `next` is just a subroutine that is provided by the Raku core.  So you can have multiple references to `next` in the same block.
```
say (1..12).map({
    next unless $_ %% 2;  # not divisible by 2, next!
    next unless $_ %% 3;  # not divisible by 3, next!
    $_                    # accept
}); # (6 12)
```

By using `next` you can create quite complicated logic when doing any `.map`ping.

## For the last time

Sometimes you want to not accept any more values in a `.map` when a certain condition fires, for instance when a certain value is seen.  Let's take one of the above examples, and make it stop when the value **7** has been encountered:
```
my $done = False;            # create flag
say (1..12).map({
    $done = True if $_ == 7; # switch flag if appropriate
    next if $done;           # we're done, next!
    next unless $_ %% 2;     # not divisible by 2, next!
    $_                       # accept
}); # (2 4 6)
```

As you can see, this is a bit of a hassle.  Fortunately, the Raku Programming Language has a solution for that in the form of the [`last` control flow statement](https://docs.raku.org/syntax/last).  So let's rewrite this example using `last`:
```
say (1..12).map({
    last if $_ == 7;     # we're done
    next unless $_ %% 2; # not divisible by 2, next!
    $_                   # accept
}); # (2 4 6)
```

Like `next`, `last` will interrupt the normal flow of the program.  When it executes, it will look up the call stack and instruct the first handler capable of handling `last` to stop iterating.  In this case, the `.map` has such a handler.

Wow, that is so much easier!

## Side effects

The previous example had one interesting side-effect: setting a flag **outside** of the block inside the `.map`.  Yes, in the Raku Programming Language you can refer to variables outside of its lexical scope, as long as they are lexically "visible".  You can use this feature for instance, to keep a count of even numbers you've seen:
```
my $seen = 0;             # initialize counter
say (1..12).map({
    next unless $_ %% 2;  # not divisible by 2, next!
    $seen++;              # increment counter
    $_                    # accept
}); # (2 4 6 8 10 12)
say "$seen even numbers"; # 6 even numbers
```

As you can see, the Raku Programming Language also as a [`++` postfix operator](https://docs.raku.org/routine/++#(Operators)_postfix_++) for incrementing integer values!

But what if you're only interested in the **how many** even numbers were seen, and not interested in the actual numbers themselves?  Well, that should be easy: remove the `say`, and the final `$_` in the block (as we're not interested in the actual value when returning from the block anyway).
```
my $seen = 0;             # initialize counter
(1..12).map({
    next unless $_ %% 2;  # not divisible by 2, next!
    $seen++;              # increment counter
});
say "$seen even numbers"; # 6 even numbers
```

And in that case, we might as well make the increment conditional, and lose the `next`!
```
my $seen = 0;             # initialize counter
(1..12).map({
    $seen++ if $_ %% 2;   # divisible by 2, increment!
});
say "$seen even numbers"; # 6 even numbers
```

This has now become a case in which the `.map` is **only** executed for its side-effects.  And actually, the Raku Programming Language has a better syntax for that: the [`for` control statement](https://docs.raku.org/syntax/for):
```
my $seen = 0;             # initialize counter
for 1..12 {
    $seen++ if $_ %% 2;   # divisible by 2, increment!
}
say "$seen even numbers"; # 6 even numbers
```

Yes.  The `for` loop in Raku, is basically a `.map` of which the body is only executed for its side-effects.  And actually, they both use the same underlying iterator mechanism.  Which means that you can use `next` and `last` also in `for` loops, because it is basically a `.map` (or vice-versa, depending on how you look at it).

The underlying iterator mechanism is material for a whole separate set of blog posts, so I won't go further into that here and now.  Suffice to say that Raku attempts to unify many different concepts that appear to be different on the surface, to deeper unifying logic and syntax.

## Signature features

Remember that in the first post of this series, we saw that you could create a block taking a value and put it into a specific variable:
```
-> $number { $number %% 2 }
```

Would you be able to use that same syntax with `for`?  Yes, you can:
```
my $seen = 0;                # initialize counter
for 1..12 -> $number {
    $seen++ if $number %% 2; # divisible by 2, increment!
}
say "$seen even numbers";    # 6 even numbers
```

In fact, the `-> $number` syntax is a property of the *block*, not of the `.map` or the `for` loop!  In fact, that feature is called the [signature](https://docs.raku.org/routine/signature) property of the block.  Does this also imply that you can use that syntax in for instance an `if` statement?  Yes, you can:
```
if complicated-calcution($input) -> $result {
    say "Result of calculation: $result";
}
```

Because that is also just a block, just as it was with `.map` or `for`!

## Conclusion
This concludes the fifth part of the series, this time introducing the `next` and `last` loop control flow statements.  And hopefully instilled the notion that a `for` loop is nothing but a `.map` that is only executed for its side-effects.  Also that blocks have signatures, that can be specified in many other situations in Raku code, such as with an `if`.

Questions and comments are always welcome.  You can also drop into the [#raku-beginner](https://web.libera.chat/?channel=#raku-beginner) channel on Libera.chat, or on Discord if you'd like to have more immediate feedback.

I hope you liked it! Thanks again for reading all the way to the end.
