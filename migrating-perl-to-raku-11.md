# Sigils (Part 2 of 2)
In this blog post we will continue to look at the subtle differences in [sigils](https://en.wikipedia.org/wiki/Sigil_(computer_programming)) (the symbols at the start of a variable name) between Perl and Raku.

## An overview
Let's repeat the overview of sigils in Perl and Raku:
```
  Sigil   Perl         Raku
  --------------------------------
    @     Array        Positional
    %     Hash         Associative
    &     Subroutine   Callable
    $     Scalar       Item
    *     Typeglob     n/a
  --------------------------------
```
# $ (Scalar vs. Item)
Compared to the `@`, `%` and `&` sigils, the `$` sigil is a bit bland. It doesn't enforce any type checks, so you can bind it to any type of object.

Therefore, when you write:
```
# Raku
my $answer = 42;
```
something like this happens:
```
# pseudo code
my $answer := Scalar.new(value => 42);
```
except at a very low level. Therefore, this code won't work, in case you wondered. And that's all there is to it when you're declaring scalar variables.

In Raku, the `$` sigil also indicates that whatever is in there, should be considered a *single* item.

So, even if a scalar container is filled with an `Array` object, it will be considered a single item in situations where iteration is required:
```
# Raku
my @foo = 1,2,3;
my $bar = Array.new(1,2,3);  # alternately: [1,2,3]
.say for @foo;  # 1␤2␤3␤
.say for $bar;  # [1 2 3]
```
Note that the latter case does only one iteration vs. three in the former case. You can indicate whether you want something to iterate or not by prefixing the appropriate sigil:
```
# Raku
.say for $@foo;  # [1 2 3] : consider the array as an item
.say for @$bar;  # 1␤2␤3␤  : consider scalar as a list
```
But maybe this brings us too far into line-noise land. Fortunately, there are also more verbose equivalents:
```
# Raku
.say for @foo.item;  # [1 2 3] : consider the array as an item
.say for $bar.list;  # 1␤2␤3␤  : consider the scalar as a list
```

# * (Typeglobs)
As you may have noticed, Raku does not have a `*` sigil nor the concept of "typeglobs". If you don't know what typeglobs are, you don't have to worry about this. You can get by very well without having to know the intricacies of symbol tables in Perl (and you can skip the next paragraph).

In Raku, the sigil is part of the name stored in a symbol table, whereas in Perl the name is stored without the sigil. For example, in Perl, if you reference `$foo` in your program, the compiler will look up "foo" (without sigil), then fetch the associated information (which is an array), and look up what it needs at the index for the `$` sigil.

In Raku, if you reference `$foo`, the compiler will look up "$foo" and directly use the information associated with that key.

Please do not confuse the `*` used to indicate slurpiness of parameters in Raku with the typeglob sigil in Perl — they have nothing to do with each other.

# Sigilless variables
Perl does not support variables without sigil out of the box (apart from maybe left-value subroutines, but that would be very clunky indeed).

Raku does not directly support sigilless variables either, but it does support binding to sigilless names by prefixing a backslash (`\`) to the name in a definition:
```
# Raku
my \the-answer = 42;
say the-answer;  # 42
```
Since the right-hand side of the assignment is a constant, this is basically the same as defining a constant:
```
# Perl
use constant the_answer => 42;
say the_answer;  # 42
```
```
# Raku my constant the-answer = 42;
say the-answer;  # 42
```
It's more interesting if the right-hand side of a definition is something else. Something like a container! This allows for the following syntactic trick to get sigilless variables:
```
# Raku
my \foo = $ = 41;                 # a sigilless scalar variable
my \bar = @ = 1,2,3,4,5;          # a sigilless array
my \baz = % = a => 42, b => 666;  # a sigilless hash
```
This basically creates nameless lexical entities (a scalar, an array, and a hash), initialises them using the normal semantics, and then binds the resulting objects (a `Scalar` container, an `Array` object, and a `Hash` object) to the sigilless name, which you can use as any other ordinary variable in Raku.
```
# Raku
say ++foo;     # 42
say bar[2];    # 3
bar[2] = 42;
say bar[2];    # 42
say baz<a b>;  # (42 666)
```
Of course, by doing this you will lose all the advantages of sigils, specifically with regard to interpolation. You will then always need to use `{ }` in interpolation.
```
# Raku
say "The answer is {the-answer}.";  # The answer is 42.
```
The equivalent is more cumbersome in most versions of Perl:
```
# Perl
say "The answer is @{[the_answer]}.”; # The answer is 42.
```

# Summary
The `$` sigil does not enforce any type constraint, differently from the `@`, `%` and `&` sigils.

The `@` and `$` prefixes indicate listification and itemisation, respectively, although it's probably more readable to use the `.list` and `.item` methods instead.

With a few syntactic tricks, you can program Raku *without* using any sigils in variable names.
