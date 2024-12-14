# Matching Maps

Lizzybel was again walking through the corridors of North Pole Grand Central and was stopped by Nanunanu, one of the IT elves, with a face a little paler than usual.  "So what is the problem?" Lizzybel asked.

Nanunanu took a deep breath and started: "...so we have built this sorta in-memory database in a Raku hash, like a key/value store.  But now we need to be able to do regex matching on the keys, and Raku hashes don't do that by themselves.  And looping over the keys and matching them one at a time is just taking too long.  And now we're looking at a deadline, and the department elf is getting all upset, and the holidays are coming nearer and nearer, and we don't know what to do".  Actually, Nanunanu's stream of consciousness went on for a little while more, but you all get the picture!

When Nanunanu drew a breath again, Lizzybel asked: "Well, that's quite a pickle you got your and your team in, have you tried hypering?".  Nanunanu looked perplexed.  "Hypering?  Isn't that like super complicated?  But no, we haven't tried that" while getting a little hope that there would be a way out of this stressful situation.

"Ok, let's do some testing", Lizzybel said while she grabbed her notebook.  "I assume that this hash is actually mostly static?" she asked, and Nanunanu nodded.  Lizzybel then started to write some code:
```raku
my %h is Map = "words".IO.lines.map: { $_ => $_ }
say "Found %h.elems() words";
{
    LEAVE say now - ENTER now;
    say %h.keys.grep: { / foo $ / }
}
```
Showing this to Nanunanu she said: "I always keep a local copy of the 'words' file in my home directory, just for these things."

And then continued: "So the first line just creates a [`Map`](https://docs.raku.org/type/Map) with all the words.  Then it shows how many words we have.  The `LEAVE` phaser and the `ENTER` phaser are used to give us the time that was spent in the lookup.  And the last line in the block does the actual work, grepping through the keys of this Map looking for any keys that end with *foo*."

Lizzybel then ran the program, and it said:
```
Found 235976 words
(foo mafoo)
0.446319075
```
"Ok, that's a good beginning" said Nanunanu with the color starting to return to their face.

"Yes, it is, but this can be made faster, because a bare `/ foo $ /` is just doing too much work if you just want to see whether it matched.  For that, you can also use [`.contains`](https://docs.raku.org/type/Str#method_contains)":
```raku
my %h is Map = "words".IO.lines.map: { $_ => $_ }
{ 
    LEAVE say now - ENTER now;
    say %h.keys.grep(*.contains(/ foo $ /);
}   
```
which showed:
```
(mafoo foo)
0.356022428
```
"The trick there is that `.contains` does not need to create a relatively expensive [`Match`](https://docs.raku.org/type/Match) for each check, it just needs to return `True` or `False`", Lizzybel explained.  "And that's already about 25% faster" said Nanunanu, showing off her amazing math skills.

"Now, if you have more than one CPU, you can use all of the other ones as well, which might make things faster, but will cost more energy because of overhead of splitting up the work over multiple CPUs and then gathering the results again.  And you only need to add *.hyper* at the right place" Lizzybel said while moving the cursor to the right place and entering just that.
```raku
my %h is Map = "words".IO.lines.map: { $_ => $_ }
{ 
    LEAVE say now - ENTER now;
    say %h.keys.hyper.grep(*.contains(/ foo $ /);
}   
```
Running that program showed:
```
(mafoo foo)
0.170500436
```
"WOW" shouted Nanunanu, "that's more than twice as fast!".  "Yup", Lizzybel agreed, "but it can get even better.  Because there's this module called [`Map::Match`](https://raku.land/zef:lizmat/Map::Match)", and off she went with:
```
$ zef install Map::Match
===> Searching for: Map::Match
===> Staging Map::Match:ver<0.0.8>:auth<zef:lizmat>
===> Staging [OK] for Map::Match:ver<0.0.8>:auth<zef:lizmat>
===> Testing: Map::Match:ver<0.0.8>:auth<zef:lizmat>
===> Testing [OK] for Map::Match:ver<0.0.8>:auth<zef:lizmat>
===> Installing: Map::Match:ver<0.0.8>:auth<zef:lizmat>
```
"That's a special version of `Map` where the keys can be regular expressions. Let's see how that works out" Lizzybel continued, smiling.
```raku
use Map::Match;
my %h is Map::Match = "words".IO.lines.map(* => 1);
{
    LEAVE say now - ENTER now;
    say (%h{/ foo $ /}:k);
}
```
which showed:
```
(mafoo foo)
0.063553282
```
"Ooh, that's cool: still 2.5x as fast as the hypered version", Nanunanu shouted out while clapping their hands.  "Yes it is" Lizzybel thought, "that was a nice piece of work I did there.  And it didn't even needed hypering under the hood, just a little NQP cheating".

Nanunanu quickly went back to their team in the knowledge that they might be able to make things up to 7 times faster.  "Thanks Lizzybel", they said while running the corridors of North Pole Grand Central.

And all was well.
