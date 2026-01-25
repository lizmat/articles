# Hello Goodbye

> This is part three in the ["Cases of UPPER"](https://dev.to/lizmat/series/35190) series of blog posts.

The first two parts discussed the four phasers linked to execution stages of the [Raku Programming Language](https://raku.org): [`BEGIN`](https://docs.raku.org/syntax/BEGIN), [`CHECK`](https://docs.raku.org/syntax/CHECK), [`INIT`](https://docs.raku.org/syntax/INIT) and [`END`](https://docs.raku.org/syntax/END).

This part will discuss the phasers related to scopes (aka, the code between curly braces): [`ENTER`](https://docs.raku.org/syntax/ENTER) and [`LEAVE`](https://docs.raku.org/syntax/LEAVE) and their special versions.

## Scope counterparts

The `ENTER` and `LEAVE` phasers are the "scope" counterparts of the `INIT` and `END` phasers.  A simple but contrived example:
```raku
if 42 {
    LEAVE say "goodbye";
    ENTER say "hello";
}
```
will show:
```
hello
goodbye
```
as one might have expected.

However there are subtle difference in behaviour between the `LEAVE` and `END` phaser with regards to [`exit`](https://docs.raku.org/routine/exit):
```
$ raku -e 'LEAVE say "goodbye"'
goodbye

$ raku -e 'LEAVE say "goodbye"; die "urgh"'
goodbye
urgh
  in block <unit> at -e line 1

$ raku -e 'LEAVE say "goodbye"; exit 42'

$ echo $?
42
```
Note that `exit` will **not** execute the `LEAVE` phaser.  But leaving a scope normally or with an execution error **will** execute the `LEAVE` phaser.

> One may have noticed that there are no scopes mentioned (`{ }`) in these examples.  That's because the mainline of any Raku program is an implicit scope.  You can think of the source of a Raku program to be between curly braces, and followed by an `exit`: `{ your program }; exit`.

As with `INIT`, the `ENTER` phasers are executed in the order they're seen.  And the `LEAVE` phasers are executed in the **reverse** order.
```
$ raku -e 'ENTER say 1; ENTER say 2'
1
2

$ raku -e 'LEAVE say 1; LEAVE say 2'
2
1
```
Although it must be said that rarely does one need to have multiple `ENTER` or `LEAVE` phasers in the same scope.

As with the `INIT` phaser, the `ENTER` phaser returns the value last seen.  So you can also use this for timing spent in a scope:
```raku
sub frobnicate() {
    LEAVE say "Frobnicated for { now - ENTER now } seconds.";
    sleep .5;  # mimic activity
}
frobnicate;
```
This would show something like:
```
Frobnicated for 0.503981515 seconds.
```

## Pre and Post Conditions

The [`PRE`](https://docs.raku.org/syntax/PRE) and [`POST`] phasers are special cases of the `ENTER` and `LEAVE` phasers.  The code in the `PRE` and `POST` phasers are supposed to produce a `True` or `False` value.  If they produce a `False` value, then an execution error will occur stating the condition.

Maybe a (again contrived) example is easier to grasp:
```raku
sub process($io) {
    PRE $io ~~ IO && $io.defined;
    42
}
say process("foo".IO);  # 42
say process(666);       # Precondition '$io ~~ IO && $io.defined' failed
```
Note that the first invocation of `process` ran ok, and the second one failed because the argument given was not an object doing the `IO` role.

> Note that a more idiomatic way would have been an `IO:D` constraint on the `$io` parameter.

Similarly with `POST`:
```raku
sub process(IO:D $io) {
    POST $io.e;
    42
}
say process(".".IO);    # 42
say process("foo".IO);  # Postcondition '$io.e' failed
```
Note again that the first invocation ran ok, and the second one failed because the argment given was did not refer to an existing path (presumably, unless you had a file "foo" in the current directory).

> Looking at the usage of `PRE` and `POST` phasers in the ecosystem, it does look like there's not a lot use made of this feature.  Implementation of these phasers predated multi-dispatch and signature checking.  So maybe these phasers are not very useful anymore.  But are still kept for backward compatibility. 

## Commit and rollback

No, the Raku Programming Language does not have [Software Transactional Memory](https://en.wikipedia.org/wiki/Software_transactional_memory) as such.  But it has two phasers that would allow one to get pretty close: [`KEEP`](https://docs.raku.org/syntax/KEEP) and [`UNDO`](https://docs.raku.org/syntax/UNDO).

If a block as a `KEEP` or `UNDO` phaser specified, one or the other will be executed depending on wheter the return from the block was considered successful.  Two criteria are applied:
- was the block exited normally (without a raised exception)?
- does [`.defined`](https://docs.raku.org/routine/defined) produce `True` or `False` on the return value of the block?

If both criteria are `True`, then any `KEEP` phaser will be executed.  If either was `False`, then the `UNDO` phaser will be executed.  This feature could e.g. be used to `commit` or `rollback` a database statement.  Or do something else entirely.
```
sub frobnicate() {
    KEEP say "commit database transaction";
    UNDO say "rollback database transaction";
    Bool.roll  # randomly succeed / fail
}

frobnicate();  # commit database transaction
frobnicate();  # rollback database transaction
```
It's hard to come up with short examples that aren't contrived.

> The `KEEP` and `UNDO` phasers are used by some very prominent modules in the [Raku Ecosystem](https://raku.land), such as [`zef`](https://raku.land/zef:ugexe/zef), [`Red`](https://raku.land/zef:FCO/Red) and [`Net::BGP`](https://raku.land/zef:jmaslak/Net::BGP).  So they have definitely proven their worth!

## Queues

Each block has a queue for `ENTER`-like phasers (`ENTER`, `PRE`, `FIRST`) and `LEAVE`-like phasers (`LEAVE`, `POST`, `KEEP`, `UNDO`, `NEXT` and `LAST`).  The `ENTER`-like phasers are executed in the order they are seen.  The `LEAVE`-like phasers are executed (if appropriate) in the **reverse** order they are seen.

The `FIRST`, `NEXT` and `LAST` phasers will be discussed in the next episode.

## Conclusion

The `ENTER` and `LEAVE` phasers apply to the scope in which they occur, and can be used for resource management and performance logging.  Just like `INIT` and `END`, but with a finer scope.

The `KEEP` and `UNDO` phasers are executed depending on whether the scope in which they occur is left successfully (`KEEP`) or unsuccessfully (`UNDO`).

The `PRE` and `POST` phaser provide gate-keeping functions for scopes, but have lost a lot of their usefulness because of other ways of gate-keeping scopes using signatures.

It doesn't matter where the phasers are specified: they will be executed at the expected moment and with the scope in which they are specified.

This concludes the third episode of cases of UPPER language elements in the Raku Programming Language.  Stay tuned for more!
