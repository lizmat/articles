# Last Close, and Quitting

> This is part five in the ["Cases of UPPER"](https://dev.to/lizmat/series/35190) series of blog posts, describing the Raku syntax elements that are completely in UPPERCASE.

This part will discuss the phasers related to concurrent and async structures, and will go into some of the mechanics behind the scenes.

## Concurrency and async

If you're unaware of the concurrency and async features of the [Raku Programming Language](https://raku.org) (especially about what [`supply` / `emit`](https://docs.raku.org/language/control#supply/emit) and `react` / `whenever` do), it might be a good idea to read up on the [Supplies section](https://docs.raku.org/language/concurrency#Supplies) of the Raku documentation on [Concurrency](https://docs.raku.org/language/concurrency).

## LAST and CLOSE

We have seen in the [previous episode](https://dev.to/lizmat/first-next-then-last-25c5) that one can have a [`LAST` phaser in loop structures](https://docs.raku.org/language/phasers#LAST).  But one can also have a [`LAST` phaser in supplies](https://docs.raku.org/language/phasers#LAST_0).

In fact, there are [three asynchronous phasers](https://docs.raku.org/language/phasers#Asynchronous_phasers): [`LAST`](https://docs.raku.org/language/phasers#LAST_0), [`CLOSE`](https://docs.raku.org/language/phasers#CLOSE) and [`QUIT`](https://docs.raku.org/language/phasers#QUIT).

- `LAST` - as soon as a supply is done / `last` is executed
- `CLOSE` - as soon as `supply` / `react` is done
- `QUIT` - when an exception occurs in the supplier

Let's start with an example with `LAST` and `CLOSE`.  We create a `supply` that will produce 5 values, and a `react` block with a single `whenever` that will handle the values produced by the `supply`.

> This example is completely synchronous because the `supply` keyword creates an "on-demand" supply, which isn't really "event" processing.  However, any other supply (e.g. a "live" supply created by the [`signal`](https://docs.raku.org/type/Supply#sub_signal) would be handled in the same way (and that *would* be event processing).
```raku
my $supply = supply {
    CLOSE say "Supply was closed.";
    emit($_) for ^5;
}

react {
    CLOSE say "Shutting down react.";
    whenever $supply {
        .say;
        LAST say "Supply is done.";
    }
}
say "React has been shut down";
```
will show:
```
0
1
2
3
4
Supply is done.
Supply was closed.
Shutting down react.
React has been shut down.
```
Note that the `LAST` phaser inside the `whenever` fires *before* the `CLOSE` phaser in the `supply`.  That's because the `supply` needs to inform each tap that it is done, before it can shutdown itself.

Also note that when the last (in this case only) `whenever` is done in a `react` block, that the `react` block will end and the code following it will be executed.

But what now if an execution error happens in the `supply` when it is producing values?  That's when the `QUIT` phaser becomes handy:
```raku
my $supply = supply {
    CLOSE say "Supply was closed.";
    for ^5 {
        die "Urgh" if $_ == 2;
        emit($_);
    }
}

react {
    CLOSE say "Shutting down react";
    whenever $supply {
        .say;
        LAST say "Supply is done.";
        QUIT {
            when X::AdHoc {
                say "Supply Error: $_.message()";
            }
        }
    }
}
say "React has been shut down";
```
will show:
```
0
1
Supply was closed.
Supply Error: Urgh.
Shutting down react.
React has been shut down.
```
Note that the `LAST` phaser will **not** fire when an execution error occurs in the `supply`, only the `QUIT` phaser will execute.  Also note that the `X::AdHoc` execution error is what `die` creates and throws, and that the `when` will cause the execution error to be disarmed.  Any other execution error will be propagated.

For instance, replacing the line `die "Urgh" if $_ == 2;` by `42[1] if $_ == 2;`, will cause this output:
```
0
1
Supply was closed.
Shutting down react.
A react block:
  in block <unit> at foo line 9

Died because of the exception:
    Index out of range. Is: 1, should be in 0..0
```
Note that in that case, execution will **not** continue after the `react` block because the execution error was not disarmed (because it was not an `X::AdHoc`).

> Yes, the use of `42[1]` may be weird, but it is one of the quickest ways to create an execution error that is not an `X::AdHoc`.  This is an error because any value in Raku can be considered a one-element list.  So `foo[0]` will return `foo`, but `foo[1]` will cause an out-of-bounds error.

## Under the hood

All of the block related phasers that have been discussed so far are stored as additional information in the [`Block`](https://docs.raku.org/type/Block) object.  Basically when the Raku grammar is parsing your code and it sees something that looks like a phaser, it just adds that to the surrounding block (represented by a `Block` object).  Then later, when the situation is right, the runtime inspects the `Block` object for any phasers that are applicable in that situation by name.  And if found, executes them.

This results in the (possibly unexpected) situation that you are allowed to specify a phaser that will never be fired in that context.  For instance:
```raku
{
    CLOSE say "closed";
    say "inside block";
}
```
will just show `inside block`, simply because the `CLOSE` phaser only makes sense inside an concurrent / async structure such as a `supply`.

> One could argue that the above example should be a compilation error, as the `CLOSE` phaser will never be called.  However, it is pretty difficult to be 100% sure at compile time how a block will actually be used in the code, and what phasers could be called.  And from a performance point of view, checking for not-applicable phasers at runtime would incur a significant performance penalty.  So it was decided to not do that and consider this way of improperly using of phasers a case of DIHWIDT (aka Doctor, It Hurts When I Do This.  "so don't do that then").

However, it *is* possible to introspect `Block` objects in Raku, because everything in Raku is an object (or can be thought of as one).  Consider this example:
```raku
my $block = {
    CLOSE say "closed";
    say "inside block";
}
$block();                           # inside block
$_() for $block.phasers("CLOSED");  # closed
```
Note that by adding `()` after the variable, you're telling Raku to *execute* the contents of that variable without giving any arguments.

The `Block.phasers` method takes the name of a block-oriented phaser and returns a `List` of `Block` objects representing each phaser seen by that name (usually one, but there can be more than one).

Now, from a performance point of view it might not be a good idea to do a lookup in the `Block` information for a phaser by a given name if there are no phasers associated with the block at all.  Fortunately the `Block` object also has a `has-phasers` method that provides an efficient way to check that.
```raku
my $block = {
    CLOSE say "closed";
    say "inside block";
}
$block();                              # inside block
if $block.has-phasers {
    $_() for $block.phasers("CLOSE");  # closed
}
```
These features allow developers to handle special features for blocks on top of what Raku already provides.

Unfortunately there's no way (yet) to add arbitrarily named phasers in Raku code: this will probably require creating a ["slang"](https://docs.raku.org/language/slangs).

## Conclusion

The `LAST`, `CLOSE` and `QUIT` phasers can be used in `supply`, `react` and `whenever` blocks and will be called at the appropriate times.

Under the hood block-oriented phasers are stored as additional information in the surrounding `Block` object.  They can be introspected with the `has-phasers` and `phasers` methods.

This concludes the fifth episode of cases of UPPER language elements in the Raku Programming Language.  Stay tuned for more!
