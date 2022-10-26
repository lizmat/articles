# Don't fear the grepper! (3)

This blog post contains the third instalment of the Don't fear the grepper! series ([Part 1](https://dev.to/lizmat/dont-fear-the-grepper-1-1k3e), [Part 2](https://dev.to/lizmat/dont-fear-the-grepper-2-4ki5)), recommended reading if you haven't already.

## Setting the topic

In the first instalment, we saw that a subroutine could be reduced to a [pointy block](https://docs.raku.org/language/functions#index-entry-pointy_blocks):
```
sub is-even($number) { $number %% 2 }
```
And that could be simplified to:
```
-> $number { $number %% 2 }
```
and after that, it got simplified to
```
* %% 2
```
Now, this is all very cool, but it limits to what you can do in the condition.  A more complex condition could not be specified that way.  So maybe we need to look at simplification in another direction as well.

Another way to simplify the pointy block version is using [the topic variable `$_`](https://docs.raku.org/language/variables#index-entry-topic_variable).
```
-> $_ { $_ %% s }
```
The topic variable `$_` is an important tool to avoid repeating yourself.  Like in any conversation, one does not need to repeat the topic over and over again.  However, you might say, in the above example we *did* repeat `$_`!  Well, in fact, you don't have to:
```
{ $_ %% 2 }
```
is an even more simplified block (which may be familiar to some of you as a ["lambda"](https://en.wikipedia.org/wiki/Anonymous_function)).  Any block with code in it, will automatically set the topic variable `$_` *inside* of that block to the argument passed to it.  One could argue that it no longer looks like a pointy block, and you'd be right.  But under the hood, it still acts as if `-> $_` was specified, and thus preserves its "pointy" behaviour.

Note that you can call still call that block as if it were a subroutine:
```
say { $_ %% 2 }(137); # False
say { $_ %% 2 }(42);  # True
```

## More on the topic

You don't need to define the `$_` variable: it is automatically defined in any scope, so you can assign to it whenever you want to.
```
$_ = 42;
say $_; # 42
```

To make this clearer in code, there is actually a [`given` control construct](https://docs.raku.org/syntax/given) that will set `$_` inside the scope it indicates:
```
given 42 {
    say $_; # 42
}
```
*Every* scope has its own topic, its own `$_` variable.  You could think of it like every scope has a `my $_` in it.  Fortunately, you don't have to specify that, so you don't have to repeat yourself!

Here's an example of multiple scopes and topics:
```
$_ = 666;
given 42 {
    say $_; # 42
}
say $_; # 666
```
Note that the `$_` after the `given` block retained the value it had before the `given` block.  No need to worry about stepping on each others toes!

## The invisible invocant

Like many things in the Raku Programming Language, there is also a method version of [`say`](https://docs.raku.org/routine/say#(Mu)_method_say).  So you could write the above code as:
```
given 42 {
    $_.say; # 42
}
```
One of the superpowers of the topic variable is that it is also the default for the invocant in method calls.  This means that if the invocant of a method call is the topic, then you don't need to specify it.  This allows you to write the previous example as:
```
given 42 {
    .say; # 42
}
```
No invocant specified: the topic is assumed.

## Expanding the notion of grep

The `grep` method allows one to filter a list of values: either a value gets through, or it does not.  In this way, the functionality of `grep` is rather limited.  What if you would not only like to filter out unwanted values, but also would like to *adapt* acceptable values on the fly?  With the `map` method, you can!

The `map` method provides a superset of the functionality of `grep`, but you can also use it as `grep` provided you're using a piece of code to do the filtering.

## Using map as a grep

Let's go back to the original example of `grep` in this series:
```
say (1..10).grep(* %% 2); # (2 4 6 8 10)
```

This would look like this when using `map`:
```
say (1..10).map({ $_ if $_ %% 2 }); # (2 4 6 8 10)
```

## Conclusion
This concludes the third part of the series, this time introducing the `map` method.

Questions and comments are always welcome.  You can also drop into the [#raku-beginner](https://web.libera.chat/?channel=#raku-beginner) channel on Libera.chat, or on Discord if you'd like to have more immediate feedback.

I hope you liked it! Thanks again for reading all the way to the end.
