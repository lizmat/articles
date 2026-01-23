# INIT to an END

> This is part two in the ["Cases of UPPER"](https://dev.to/lizmat/cases-of-upper-imn) series of blog posts.

The first part of this series discussed the two [phasers](https://docs.raku.org/language/phasers) in the [Raku Programming Language](https://raku.org) of which the code is executed at compile time: `BEGIN` and `CHECK`.

This part will discuss the two phasers that are directly connected with the runtime of a Raku program: `INIT` and `END`.

## INIT

The [`INIT`](https://docs.raku.org/syntax/INIT) phaser is executed **before** any actual execution of a program.

When seen in the code during compilation, the `Block` or `thunk` will be added to a queue of code to be executed.  Once all bytecode of a program is ready for execution, all `INIT` phasers seen (both in the source of the program being run, but *also* any `INIT` phasers in any of the modules that have been `use`d) will be executed in the order they were seen or loaded:
```perl
INIT say "first";
INIT say "second";
```
will show:
```
first
second
```

As with the `BEGIN` and `CHECK` phasers, the last value produced will be returned by the phaser where it was specified.  So if you want to know how long your program has been running *in* your program, you can use the following construct using [`now`](https://docs.raku.org/routine/now):
```perl
sleep .5;           # mimic activity
say now - INIT now; # difference between now and when INIT ran
```
which will show something like:
```
0.503897046
```

`INIT` phasers are typically used to set up (connections to) external resources that are always needed for the execution of a program, such as making a connection to a database.
```perl
INIT my $dbh = DB::xxx.connect(...);
```
Because a thunk was used, the definition of `$dbh` is in the surrounding scope.  Which will make it accessible to for instance an `END` phaser.

### END

The [`END`](https://docs.raku.org/syntax/END) phaser is executed **after** execution of a program is finished (either from just running off the end, or by an execution error, or by executing an [`exit`](https://docs.raku.org/routine/exit)).

When seen in the code during compilation, the `Block` or `thunk` will be added to a queue of code to be executed.  All `END` phasers seen (both in the source of the program being run, but *also* any `END` phasers in any of the modules that have been [`use`](https://docs.raku.org/syntax/use)d) will be executed in the **reverse** order they were seen or loaded:
```perl
END say "first";
END say "second";
```
will show:
```
second
first
```
The `END` phaser does **not** return any value, because effectively execution of the program has stopped, so there is little point.

`END` phasers are typically used to perform an orderly shutdown of external resources.  For instance:
```perl
END .disconnect with $dbh;
```
This will disconnect the database connected to in `$dbh` *if* a connection was actually made.  The reason this is guarded by a [`with`](https://docs.raku.org/syntax/with%20orwith%20without), because an error *can* have occurred in an earler `INIT` phaser, so that `$dbh` was never initialized.  Otherwise you would get an execution error in the `END` phaser because `Any` doesn't have a `.disconnect` method, which would be annoying.

> Note that even though Raku has support for a [`DESTROY`](https://docs.raku.org/syntax/DESTROY) method, there is no guarantee that that method will actually ever be executed.  In other words: the Raku Programming Language does **not** support timely destruction.  Which makes proper shutdown of external resources using phasers important if you do want timely management of such resources.

Another use of an `END` phaser, is to report statistics about the execution of a program.  A very basic example would be:
```perl
END say now - INIT now;
```
which would show the number of seconds the program actually ran.

Or to keep a log of when programs were executed a slightly more complicated `END` phaser:
```perl
END "log".IO.spurt(:append,
  DateTime.now ~ ": $*PROGRAM-NAME ran for { now - INIT now } seconds\n"
);
```
which would add a line (hence the `:append`) for every program run to a "log" file with the date and time of when the run ended, the program name, and how many seconds it ran.

> Note the use of [`$*PROGRAM-NAME`](https://docs.raku.org/language/variables#$*PROGRAM-NAME), another all upper case element in the Raku Programming Language that will be part of a future episode in this series.

## Scoping

Although all of the above examples show the use of the `INIT` and `END` phasers at the top level of the code, there is no reason why you couldn't put an `INIT` or an `END` phaser inside a class, method or subroutine.  Why would you do that?  Well, usually it makes sense because of the scope of the variables that are being used in the `INIT` or `END` phaser.

Take for instance a subroutine that tracks how many times it is being called, and shows the result after the program is done:
```perl
sub track() {
    state $seen;
    END say "'track' was called $seen times.";
    ++$seen;
}

track for ^10;
```
will show:
```
'track' was called 10 times.
```

## Conclusion

The `INIT` and `END` phaser apply to the execution stage (runtime) of a Raku program, and can be used for resource management and performance logging.

It doesn't matter where the phasers are specified: they will be executed at the expected moment and with the scope in which they are specified.

This concludes the second episode of cases of UPPER language elements in the Raku Programming Language.  Stay tuned for more!
