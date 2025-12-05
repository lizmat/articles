# Allowing for fewer dollars

*Lizzybel* had been taking a bit of vacation from all of the busy-ness in the corridors of North Pole Grand Central.

While doing a small visit to the corridors, she ran into *Nanunanu*, one of the IT elves.  *Nanunanu* was a bit worried, because they had not seen *Lizzybel* for a while.  "Don't worry", said *Lizzybel*.  "I'm just recharching my batteries a bit while doing some other stuff that has been neglected by me for a while.  But I have been following developments from a distance, to stay at least a bit in the loop", *Lizzybel* said with a bit of a grin.  "Ah, ok", said Nanunanu, "anything particular that caught your eye?".

"Now that you mention it: it looks like quite a few of potential users of the [https://raku.org](Raku Programming Language) are put off by the use of sigils in variable declarations, specifically the `$`", said *Lizzybel* while taking out her phone and showing a [HackerNews comment](https://news.ycombinator.com/item?id=45856851) to *Nanunanu*.

"What a silly reason to not want to look deeper into Raku".  *Nanunanu* agreed and went on their way because busy, busy, busy!

While going home, *Lizzybel* was thinking: "On the other hand, it was clear that this was about first impressions.  And first impressions are important.  So because of this first impression, the Raku Programming Language was potentially missing out on a significant number of new users!  What a pity!"

## A constant

But the Raku Programming Language is no stranger to sigilless constants:
```raku
my constant answer = 42;
```
"is but an example", Lizzybel was thinking to herself.  "And with a little trick, you could even make sigilless variables", she was mumbling to herself:
```raku
my \answer = my $;
answer = 42;
say answer;  # 42
```
But that is really yucky.  And would not help with a first impression of the Raku Programming Language, at all!

## Emojional

Then she was reminded of a playful module she'd made several years ago: [Slang::Emoji](https://raku.land/zef:lizmat/Slang::Emoji).  It allowed one to define and use variables whose name was a single character emoji, such as `👍` or `🏳️‍🌈`:
```raku
use Slang::Emoji;
my 👍 = 42;
say 👍;  # 42
```
To make this possible, she remembered that she had actually sneaked in a special `token` into the grammar of the Raku programming Language to be able to do this: [sigilless-variable](https://github.com/rakudo/rakudo/blob/main/src/Perl6/Grammar.nqp#L1825).  Maybe that `token` could be used to create sigilless variables in Raku as well?

## Nogil

Turns out there had been a Raku slang for sigilless variables [already](https://github.com/tinmarino/nogil?tab=readme-ov-file#slangnogil) by *Martin Tourneboeuf*.  But sadly that had bitrotted.  "Why not use that namespace?", Lizzybel thought to herself.

The initial iteration of transmogrifying the `Slang::Emoji` module into `Slang::Nogil` looked simple enough.  Just replace `<.:So>` with `<.ident>+`, and add a check that we're actually in a definition (`$*IN_SPEC`), and voila: [Slang::Nogil 1.1](https://raku.land/zef:lizmat/Slang::Nogil/changes?v=1.1).
```
use Slang::Nogil;
my answer = 42;
say answer;  # 42
```
But then some issues started to become clear(er).

## Nogil vs Emoji

Because both `Slang::Emoji` and `Slang::Nogil` mix in a new version of the "sigilless-variable" `token`, one module was trampling on the other.
