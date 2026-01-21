# INIT to an END

> This is part two in the ["Cases of UPPER"](https://dev.to/lizmat/cases-of-upper-imn) series of blog posts.

The first part of this series discussed the two [phasers](https://docs.raku.org/language/phasers) in the [Raku Programming Language](https://raku.org) of which the code is executed at compile time: `BEGIN` and `CHECK`.

This part will discuss the two phasers that are directly connected with the runtime of a Raku program: `INIT` and `END`.

## INIT

The [`INIT`](https://docs.raku.org/syntax/INIT) phaser is executed **before** any actual execution of a program.

When seen in the code during compilation, the `Block` or `thunk` will be added to a queue of code to be executed.  Once all bytecode of a program is ready for execution, all `INIT` phasers seen (both in the source of the program being run, but *also* any `INIT` phasers in any of the modules that have been `use`d) will be executed in the order they were seen.
```raku
INIT say "first";
INIT say "second";
```
will show:
```
first
second
```

As with the `BEGIN` and `CHECK` phasers, the last value produced will be returned by the phaser where it was specified.  So if you want to know how long your program has been running *in* your program, you can use the following construct using [`now`](https://docs.raku.org/routine/now):
```raku
sleep .5;           # mimic activity
say now - INIT now; # difference between now and when INIT ran
```
which will show something like:
```
0.503897046
```

`INIT` phasers are typically used to set up (connections to) external resources that are always needed for the execution of a program, such as making a connection to a database.
```raku
INIT my $dbh = DB::xxx.connect(...);
```

### END

The [`END`](https://docs.raku.org/syntax/END) phaser is executed **after** execution of a program is finished (either from just running off the end, or by an execution error, or by executing an [`exit`](https://docs.raku.org/routine/exit).

When seen in the code during compilation, the `Block` or `thunk` will be added to a queue of code to be executed.  All `END` phasers seen (both in the source of the program being run, but *also* any `END` phasers in any of the modules that have been `use`d) will be executed in the **reverse** order they were seen.
```raku
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
```raku
END .disconnect with $dbh;
```
This will disconnect the database connected to in `$dbh` *if* a connection was actually made.  The reason this is guarded by a [`with`](https://docs.raku.org/syntax/with%20orwith%20without), because an error *can* have occurred in an earler `INIT` phaser, so that `$dbh` was never initialized.  Otherwise you would get an execution error in the `END` phaser, which would be annoying.

Another use of an `END` phaser, is to report statistics about the execution of a program.  A very simple example would be:
```raku
END say "$*PROGRAM-NAME ran for { now - INIT now } seconds";
```
which would show the number of seconds the program actually ran (and its name).

Or to keep a log of when programs were executed:
```raku
END "log".IO.spurt(:append,
  DateTime.now ~ ": $*PROGRAM-NAME ran for { now - INIT now } seconds\n"
);
```
which would add a line (hence the `:append`) for every program run to a "log" file with the date/time of the run ended, the program name, and how many seconds it ran.

## Conclusion

The `INIT` and `END` phaser apply to the execution stage (runtime) of a Raku program, and can be used for resource management and performance logging.

This concludes the second episode of cases of UPPER language elements in the Raku Programming Language.  Stay tuned for more!
