# The Magic of $/

Santa was dabbling a bit with regexes in the Raku Programming Language, just to satisfy their curiosity.  Trying to find out how many children have a first name that starts with a vowel.  That seemed like a nice little project!

Since the Big Database Server at the North Pole was already heating up a lot, and the amount of ice on the North Pole was already dwindling at alarming rates, Santa decided to not use the database at all for this dabbling.  Instead Santa decided to use the "words" file that you can find on Unix system in `/usr/share/dict/words`, and copied it to "kiddos".

Santa had already learned about [`Bags`](https://docs.raku.org/type/Bag) earlier in the year, and had also tried out several features of the the [regex syntax](https://docs.raku.org/language/regexes) of Raku.  And so came up with the following program:
````raku
my %b is Bag = "kiddos".IO.lines.map: {
    ~$0 if /^ (a | e | i | o | u) /;
}
for %b.sort(-*.value) {
    state $total;
    FIRST say "Names that start with vowels:";
    FIRST say "-----------------------------";
    printf "%1s: %5d\n", .key, .value;
    $total += .value;
    LAST printf "-------- +\n%8d\n", $total;
}
````
Which outputs:
````
Names that start with vowels:
-----------------------------
u: 16179
a: 14537
i:  8303
e:  7818
o:  7219
-------- +
   54056
````
Sante decided that the [`FIRST`](https://docs.raku.org/language/phasers#FIRST) and [`LAST`](https://docs.raku.org/language/phasers#LAST) phasers were a really nice way to keep the summarizing code in a single scope.  The fact that you could use a [`state variable`](https://docs.raku.org/language/variables#The_state_declarator) for that as well, even made things even nicer.

But the little program took longer to execute than Santa liked.  Even though Santa's computer was not the most modern one, it still had 4 CPUs, and only one was used in this little program.  Somewhere Santa had read about being able to execute a map in parallel.  Hmmmm, Santa mumbled.  Ah yes, [`race`](https://docs.raku.org/type/Iterable#method_race) is what they where looking for!  Just adding 5 letters should make all the difference!
````raku
#                               👇
my %b is Bag = "kiddos".IO.lines.race.map: {
    ~$0 if /^ (a | e | i | o | u) /;
}
````
But, but, that didn't work at all?  Every time the program would crash with strange, totally unexpected errors all starting with:
````
A worker in a parallel iteration (hyper or race) initiated here:
  in block ………
````
Santa decided to call in Lizzybel to tell them that they had found a bug in Raku.  Santa actually felt quite proud about that!

Lizzybel was quite taken aback by the weirdness of the errors as well.  Why would that not work?  They started digging and after a few hours, realized that there was at least a very simple workaround:
````raku
my %b is Bag = "kiddos".IO.lines.race.map: {
#   👇
    my #/;  # give this scope its own $/
    ~$0 if /^ (a | e | i | o | u) /;
}
````
Santa was really happy that such a little change would fix the issue and allow the program to run as fast as it could on Santa's computer.  But Santa also wanted to know *why* this was necessary!  So Lizzybel explained:

"As you know, matching will set the `$/` variable.  And `$0` is just short for `$/[0]`.  However, you almost never need to define a `$/` yourself, because there's one defined in every `sub`, `method` or mainline automatically for you.  But inside any other block `{ … }`, **no** `$/` will be defined for you.  This allows this common code structure to work:"
`````raku
"foo bar" ~~ / (\w+) /;
if $0 {
    say $0;  # ｢foo｣
}
`````
Lizzybel continued: "So the `$/` implicitely used with `$0` inside the `if`, refers to the `$/` that is automatically defined in the outer lexical scope (in this case, the main line of the program).  If there would have been a separate `$/` inside the block of the `if` statement, it would mask the outer `$/`, and thus show an undefined value for `$0`, like this:

`````raku
"foo bar" ~~ / (\w+) /;
if $0 {
    my $/;
    say $0;  # (Any)
}
`````
Again, Lizzybel continued: "So in Raku it was decided that `$/` would always refer to an automatically defined `$/` in any of the *lexically* outer `sub`-like scopes.  This is not an issue with any single-threaded program.  But becomes an issue when multiple threads are accessing the same `$/` simultaneously.  And that causes the strange errors that you see".

"But, couldn't this be handled automatically by Raku?  Why would **I** have to add a `my $/` there?  This feels like a bit of a trap!", Santa interjected.

"This can not be fixed very easily", said Lizzybel, and continued: "That's because the `.race` method is just like any other method in Raku, there is nothing special about it.  It produces a `RaceSeq` object which happens to have a [`.map` method](https://docs.raku.org/type/RaceSeq#method_map) that takes a block as a parameter.  What that block actually contains, is completely opaque to the executor.  The result of the `.map` just happens to have an `.iterator` method that serializes the results that are calculated in parallel."

"Hmmmm", said Santa, now almost grumbling.

Not deterred by Santa, Lizzybel continued again: "And since the solution is pretty simple, it was decided that this should really be documented as a trap.  At least until there's a smarter way to handle this situation".

"I guess I'm a bit of a sorcerer's apprentice when it comes to multi-threading", Santa mumbled under their breath.  Lizzybel heard it nonetheless and responded: "In a way we all are, Santa, is I didn't immediately grok the issue either".

After this reassurance Santa happily continued with dabbling in Raku, never too old to learn!  And all was well.
