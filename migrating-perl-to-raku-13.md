# Phasers (Part 2 of 2)
If you are only interested in learning how Perl special blocks work in Raku, you can skip this blog post.  But you will be missing out on quite a few nice and useful features that are available in Raku.

## Block and Loop phasers
Block and Loop phasers are always associated with the surrounding Block, regardless of where they are located in the Block. Except you are not limited to having just one of each — although you could argue that having more than one doesn't improve maintainability.

Note that any `sub` or `method` is also considered a Block with regards to these phasers: you could think of the `Block` as a base-class for subroutines and methods.
```
  Name    Description
  -----------------------------------------------------
  ENTER   Run every time when entering a block
  LEAVE   Run every time wjen leaving a block
  PRE     Check condition before running a block
  POST    Check return value afgter having run a block
  KEEP    Run every time a block is left successfully
  UNDO    Run every time a block is left unsuccessfully
  -----------------------------------------------------
```
## ENTER & LEAVE
The [`ENTER`](https://docs.raku.org/language/phasers#ENTER) and [`LEAVE`](https://docs.raku.org/language/phasers#LEAVE) phasers are pretty self-explanatory: the `ENTER` phaser is called whenever a block is entered. The `LEAVE` phaser is called whenever a block is left (either gracefully or through an exception).

A simple example:
```
# Raku
say "outside";
{
    LEAVE say "left";
    ENTER say "entered";
    say "inside";
}
say "outside again";
# outside
# entered
# inside
# left
# outside again
```
The last value of an `ENTER` phaser is returned so that it can be used in an expression.

Here's a bit of a contrived example:
```
# Raku
{
    LEAVE say "stayed {now - ENTER now} seconds";
    sleep 2;
}
# stayed 2.001867 seconds
```
The `LEAVE` phaser corresponds to the "DEFER" functionality in many other modern programming languages.

## KEEP & UNDO
The [`KEEP`](https://docs.raku.org/language/phasers#KEEP) and [`UNDO`](https://docs.raku.org/language/phasers#UNDO) phasers are special cases of the `LEAVE` phaser. They are called depending on the return value of the surrounding block.

If the result of calling the `defined` method on the return value is `True`, then any `KEEP` phasers will be called. If the result of calling the `defined` method is *not* True, then any `UNDO` phaser will be called. The actual value of the block will be available in the topic (`$_`) inside the phaser.
.
A contrived example may clarify:
```
# Raku
for 42, Nil {
    KEEP { say "Keeping because of $_" }
    UNDO { say "Undoing because of $_.raku()" }
    $_;  # the return value of the block
}
# Keeping because of 42
# Undoing because of Nil
```
Maybe real-life example would be clearer:
```
# Raku
{
    KEEP $dbh.commit;
    UNDO $dbh.rollback;
    …  # set up a big transaction in a database
    True;  # indicate success
}
```
So, if anything goes wrong with setting up the big transaction in the database, the `UNDO` phaser makes sure the transaction can be rolled back. Conversely, if the block is successfully left, the transaction will be automatically committed by the `KEEP` phaser.

The `KEEP` and `UNDO` phasers give you the building blocks for a poor man's [software transactional memory](https://en.wikipedia.org/wiki/Software_transactional_memory).

## PRE & POST
The [`PRE`](https://docs.raku.org/language/phasers#PRE) phaser is a special version of the `ENTER` phaser. The [`POST`](https://docs.raku.org/language/phasers#PRE) phaser is a special case of the `LEAVE` phaser.

The `PRE` phaser is expected to return a true value if it is ok to enter the block. If it does not, then an exception will be thrown. The `POST` phaser receives the return value of the block and is expected to return a true value if it is ok to leave the block without throwing an exception.

An example with `PRE`:
```
# Raku
{
    PRE {
        say "called PRE”;
        False;      # throws exception
    }
    …
}
say "we made it!";  # never makes it here
# called PRE
# Precondition '{ say "called PRE"; False }' failed
```
An example with `PRE` and `POST`:
```
# Raku
{
    PRE {
        say "called PRE";
        True;               # does NOT throw
    }
    POST {
        say "called POST";
        False;              # throws exception
    }
    say "inside the block"  # also returns True
}
say "we made it!";  # never makes it here
# called PRE
# inside the block
# called POST
# Postcondition '{ say "called POST"; False }' failed
```

If you just want to check if a block returns a specific value or type, you are probably better off specifying a return signature for the block. Note that:
```
# Raku
{
    POST {
        $_ ~~ Int;     # check if return value is an Int
    }
    …                  # calculate result
    $result;
}
```
is just a very roundabout way of saying:
```
# Raku
--> Int {  # return value should be an Int
   …      # calculate result
   $result;
}
```
In general, you would use a `POST` phaser only if the necessary checks would be very involved and not reducible to a simple type check.

## Loop phasers
```
  Name    Description
  ----------------------------------------------------------
  FIRST   Run before the first iteration
  NEXT    Run after each completed iteration, or with "next"
  LAST    Run after the last iterayion, or with "last"
  ----------------------------------------------------------
```
Loop phasers are a special type of phaser specific to loop constructs. One is run before the first iteration ([`FIRST`](https://docs.raku.org/language/phasers#FIRST)), one is run after each iteration ([`NEXT`](https://docs.raku.org/language/phasers#FIRST)), and one is run after the last iteration ([`LAST`](https://docs.raku.org/language/phasers#FIRST)).

The names speak for themselves. A bit of a contrived example:
```
# Raku
my $total = 0;
for 1..5 {
    $total += $_;
    LAST  say "------ +\n$total.fmt('%6d')";
    FIRST say "values\n======";
    NEXT  say .fmt('%6d');
}
# values
# ======
#      1
#      2
#      3
#      4
#      5
# ------ +
#     15
```
Supported loop constructs include `loop`, `while`, `until`, `repeat`, `for`, and `map` and friends.  You can use loop phasers with other block phasers if you want, but this is usually unnecessary.

## Summary
In addition to the Perl special blocks that have counterparts in Raku (called phasers), Raku has a number of special-purpose phasers related to blocks of code and looping constructs.

The Raku Programming Language also has phasers related to exception handling and warnings, event-driven programming, and document (pod) parsing; these will be covered in separate blog posts.
