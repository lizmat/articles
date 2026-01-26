# First Next, then Last

> This is part four in the ["Cases of UPPER"](https://dev.to/lizmat/series/35190) series of blog posts, describing the Raku syntax elements that are completely in UPPERCASE.

This part will discuss the phasers related to loop structures (the code between curly braces that is being repeatedly called): [`FIRST`](https://docs.raku.org/syntax/FIRST), [`NEXT`](https://docs.raku.org/syntax/NEXT) and [`LAST`](https://docs.raku.org/syntax/LAST).

## But first, a statement

The [second](https://dev.to/lizmat/init-to-an-end-30hc) episode already saw the use of the [`state`](https://docs.raku.org/syntax/state) scope identifier in one of the examples, without going into it much.  Since `state` is affecting some aspects handled in this blog post, it felt important to first circle back to it before going on.

In short, a `state` variable is a lexically scoped variable that will keep its value between invocations of the scope in which it is defined.  For example:
```raku
sub frobnicate() {
  state $times += 1;  # defined inside "frobnicate"
  END say "Frobnicated $times times.";
}
frobnicate for ^5;
```
will show:
```
Frobnicated 5 times.
```
It is really as if the variable `$times` has been defined in the outer scope.  This gives the same result:
```raku
my $times;  # defined outside of "frobnicate"
sub frobnicate() {
  $times += 1;
  END say "Frobnicated $times times.";
}
frobnicate for ^5;
```
The difference being that the `my $times` variable is visible to other pieces of the program, where as the `state $times` variable is **not**.

A slightly adapted example:
```raku
sub frobnicate() {
  for ^3 {
    state $times += 1;  # defined inside "for" loop
    END say "Frobnicated $times times.";
  }
}
frobnicate for ^5;
```
will show:
```
Frobnicated 3 times.
```
Not 5.  Not 15.  But 3.  That's because the above code could also be written as:
```raku
sub frobnicate() {
  my $times;
  for ^3 {
    $times += 1;  # defined outside of "for" loop
    END say "Frobnicated $times times.";
  }
}
frobnicate for ^5;
```
And since the `END` phaser occurs only once in the code, it will only see the value of `$times` from the last time that "frobnicate" had been called.

> The way initialization of `state` variables is organized is slightly more complex than described above.  In most cases the above explanation is correct, and when it is not you're probably getting into DIHWIDT (Doctor, It Hurts When I Do This: "so don't do that") territory.

Finally it should be noted that `state` variables, like any other non-atomic variables, are **not** thread-safe.
```raku
sub frobnicate() {
  state $times += 1;
  END say "Frobnicated $times times";
}
await (^100).map: {
  start {
    sleep rand;  # create some chaos
    frobnicate;
  }
}
```
will show fewer than 100 frobnications.

This is caused by multiple threads trying to update the `$times` variable simultaneously.  This may miss updates, because `$times += 1` is essentially `$times = $times + 1`.  If two threads fetch the value of `$times` simultaneously, then one of the threads will lose its update to `$times` because it didn't "see" the update of the other thread before fetching it.

It's important to recognize this behaviour of `state` variables, before they are being used.  However, the system uses some state variables under the hood.  So your code may be affected by this behaviour without you as a develeoper realizing that.

## Loop-di-loop

All loop structures in the [Raku Programming Language](https://raku.org) ([`for`](https://docs.raku.org/syntax/for), [`while` / `until`](https://docs.raku.org/syntax/while%20until) and [`loop`](https://docs.raku.org/syntax/loop) allow for the specification of an additional 3 types of phaser: [`FIRST`](https://docs.raku.org/syntax/FIRST), [`NEXT`](https://docs.raku.org/syntax/NEXT) and [`LAST`](https://docs.raku.org/syntax/FIRST).

The names of these phasers are pretty explanatory:
- FIRST - runs the first time a loop structure is entered
- NEXT - runs each time after an iteration is done
- LAST - runs after the last iteration is done

A simple example:
```raku
for ^5 -> $id {
  state $left  += $id;
  state $right += $id + 2;
  FIRST say "ID | +2\n===+===";
  NEXT say " $id |  { $id + 2 }";
  LAST say "===+=== +\n$left | $right";
}
```
will show:
```
ID | +2
===+===
 0 |  2
 1 |  3
 2 |  4
 3 |  5
 4 |  6
===+=== +
10 | 20
```
Using the `FIRST` and `LAST` phaser allows one to easily add a header and a footer using variables that only exist inside the loop structure.

The `NEXT` phaser runs when an iteration is ended normally, or is started with a [`next`](https://docs.raku.org/syntax/next) command.  The `LAST` phaser is run after the last iteration is done, or when leaving the scope is initiated with a [`last`](https://docs.raku.org/syntax/last) command.

The `FIRST` phaser uses an invisible `state` variable under the hood, so depends on the peculiarities of its implementation.

## FIRST in the future

In the next language level of Raku, the `FIRST` phaser will become active for **any** scope, not just inside of loop structures.  It will then also return the value produced by the `FIRST` phaser.
```
$ RAKUDO_RAKUAST=1 raku -e '{ say now - FIRST now }'
0.000048209
```

This functionality is similar to what is provided by [`once`](https://docs.raku.org/syntax/once).  But that looks phaser-like?  Why isn't `once` a phaser and thus in all uppercase?

There have been a lot of discussions about that, but in the end it was decided that it is not a phaser because it runs in-line like other statement prefixes ([decision from 2013](https://irclogs.raku.org/perl6/2013-05-30.html#04:04)).  So:
```
$ raku -e '{ say now - once now }'
-0.000055917
```
will show a negative value because the `once now` is executed **after** the first `now`.

The `FIRST` phaser however will run when the scope is entered, **not** where the `FIRST` phaser occurs in code.

## Conclusion

The `FIRST`, `NEXT` and `LAST` phasers apply to a scope that is being used in a loop structure.

The `FIRST` phaser depends on a `state` variable under the hood.

With the next language level, the `FIRST` phaser will become active for **any** scope and return the value it produced.  This is like `once` but at entry of the scope (not in-line).

This concludes the fourth episode of cases of UPPER language elements in the Raku Programming Language.  Stay tuned for more!
