# Syntactic Changes between Perl and Raku (Part 3 of 3)

## For, While and Loop
The [`while`](https://docs.raku.org/language/control#while,_until) loop structure is (almost) the same in Raku: you just don’t need to put the condition between parentheses in Raku:
```
# Perl
while ($condition) {
    …  # do your stuff
}
```
```
# Raku
while $condition {
    …  # do your stuff
}
```
In Perl there is no specific syntax for a loop structure that will iterate until the end of times (or until a [`last`](https://docs.raku.org/language/control#last) or a [`return`](https://docs.raku.org/language/control#return) is executed):
```
# Perl
while (1) {
    …  # do your stuff
    last if $condition;
}
```
In Raku there is, the [`loop`](https://docs.raku.org/language/control#loop) statement:
```
# Raku
loop {
    …  # do your stuff
    last if $condition;
}
```
In Perl, the `for` loop structure also supports C-syntax to indicate the control flow.
```
# Perl
for (my $i = 0; $i < 10; $i++) {
    say $i;
}
```
In Raku, this is not the case: the `loop` statement is also used for that:
```
# Raku
loop (my $i = 0; $i < 10; $i++) {  # "loop" instead of "for"
    say $i;
}
```
Note that this is one of the few cases in Raku where parentheses are actually required!

The syntax for aliasing the iterated value in `for` is slightly different in Raku.  In Perl, you would specify `my` with name of a lexical variable.
```
# Perl
for my $elem (@elements) {
    $elem *= 2;
}
```
In Raku this is part of the standard syntax for blocks:
```
# Raku
for @elements -> $elem {  # take a value from the array each time
    $elem *= 2;
}
```
Please note that you can also use this syntax much more generally, e.g. with an `if` statement:
```
# Raku
if complicated-calculation -> $result {
    …  # do something with $result
}
```
There is no `foreach` in Raku: please use `for` instead.  Trying so, will give you this compilation error:
```
# Raku
foreach 1,2,3 { .say }
===SORRY!=== Error while compiling …
Unsupported use of 'foreach'; in Raku please use 'for' at
-e:1 ------> foreach⏏ 1,2,3 { .say }
```
Note that Raku takes extra care to give useful error messages.  If an error message does not clearly indicate the problem, then the error message is considered LTA (Less Than Awesome) and is worthy of making an issue for.

## No Special Case Syntax
Perl has a few special syntax cases, most notably with `map`.  Raku does not have any special syntax for certain statements.  Even something like `map` in Raku, is just a subroutine that is provided by the system.

This means that something like this allowed in Perl:
```
# Perl
my @squares = map { $_ * $_ } @numbers;  # note absence of comma!
```
But since (almost) nothing is special in Raku, that same code requires a comma, because `map` is a subroutine like any other, and thus needs its arguments separated by commas:
```
# Raku                       ↓
my @squares = map { $_ * $_ }, @numbers;  # note required comma 
```

## Iterating Over Hashes
There is a subtle difference in iterating over hashes between Perl and Raku.

In Perl, iterating over a hash means producing the key and value of each element separately:
```
# Perl
my %h = (a => 42, b => 666);
for (%h) {
    say
}
# a␤42␤b␤666␤
```
In Raku, a hash is considered to be an unordered list of [`Pair`](https://docs.raku.org/type/Pair) objects.  Which you will get if you iterate over them:
```
# Raku
my %h = a => 42, b => 666;
for %h {
    .say
}
# a => 42␤b => 666␤
```
If you want the Perl behaviour, that is also possible with the [`.kv`](https://docs.raku.org/type/List#routine_kv) (for key / value) method:
```
# Raku
my %h = a => 42, b => 666;
for %h.kv {
    .say
}
# a␤42␤b␤666␤
```
If you do not want a `Pair` object in each iteration, but a separate key and value for each iteration, that is also possible by using some of the unique properties of signatures that will be discussed in more depth later:
```
# Raku
my %h = a => 42, b => 666;
for %h.kv -> $key, $value {
    say “key = $key, value = $value”;
}
# key = a, value = 42␤key = b, value = 666␤
```

## Reading Lines From STDIN
This very common idiom in Perl:
```
# Perl
while (<>) {
    chomp;
    …  # do something with $_
}
```
has been replaced by the [`lines`](https://docs.raku.org/routine/lines) sub / method in Raku:
```
# Raku
for lines() {
    …  # do something with $_
}
```
Note that even though it looks like this will first slurp the whole of STDIN, this is not the case in Raku.  Subroutines and methods can return lazy sequences (as in this case), so there will always only be one line of the file (well, approximately, considering buffering) in memory.

Also note that chomping (the removal of the newline character from the end of the line) is **on** by default in Raku.  Should you wish to not have your lines chomped, you can also switch that off by specifying the `chomp` named parameter:
```
# Raku
for lines(chomp => False) {  # also known as :!chomp
    …  # do something with $_
}
```

## Summary
Even though there are some gotchas when trying to write Raku code coming from Perl, it generally is much the same, and generally simpler with fewer exceptions.  This makes it very possible to write Raku code in a way that is very similar to Perl idioms.  Now whether that is a good thing in the long run, is another matter.
