# Allowing for fewer dollars

*Lizzybel* had been taking a bit of vacation from all of the busy-ness in the corridors of North Pole Grand Central.

While doing a small visit to the corridors, she ran into *Nanunanu*, one of the IT elves.  *Nanunanu* was a bit worried, because they had not seen *Lizzybel* for a while.  "Don't worry", said *Lizzybel*.  "I'm just recharching my batteries a bit while doing some other stuff that has been neglected by me for a while.  But I have been following developments from a distance, to stay at least a bit in the loop", *Lizzybel* said with a bit of a grin.  "Ah, ok", said Nanunanu, "anything particular that caught your eye?".

"Now that you mention it: it looks like quite a few of potential users of the [Raku Programming Language](https://raku.org) are put off by the use of sigils in variable declarations, specifically the `$`", said *Lizzybel* while taking out her phone and showing a [HackerNews comment](https://news.ycombinator.com/item?id=45856851) to *Nanunanu*.

"What a silly reason to not want to look deeper into Raku".  *Nanunanu* agreed and went on their way because busy, busy, busy!

While going home, *Lizzybel* was thinking: "On the other hand, it was clear that this was about first impressions.  And first impressions are important.  So because of this first impression, the Raku Programming Language was potentially missing out on a significant number of new users!  What a pity!"

## A constant

When back home, *Lizzybel* thought: "But the Raku Programming Language is no stranger to sigilless constants"
```raku
my constant answer = 42;
```
"is but an example".  "And with a little trick, you could even make sigilless variables", she was mumbling to herself:
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

Turns out there had been a Raku slang for sigilless variables [already](https://github.com/tinmarino/nogil?tab=readme-ov-file#slangnogil) by *Martin Tourneboeuf*.  But sadly that had bitrotted.  "Why not use that namespace?", *Lizzybel* thought to herself.  "Indeed, why not?".

The initial iteration of transmogrifying the `Slang::Emoji` module into `Slang::Nogil` looked simple enough.  Just replace `<.:So>` with `<.ident>+`, and add a check that we're actually in a definition (`$*IN_SPEC`), and voila: [Slang::Nogil 1.1](https://raku.land/zef:lizmat/Slang::Nogil/changes?v=1.1).
```
use Slang::Nogil;
my answer = 42;
say answer;  # 42
```
And fortunately, *Martin Tourneboeuf* was [happy with the result](https://github.com/tinmarino/nogil/issues/3#issuecomment-3534404648).

All was good, but then some issues started to become clear(er).

## Nogil vs Emoji

Because both `Slang::Emoji` and `Slang::Nogil` mix in a new version of the "sigilless-variable" `token`, one module was trampling on the other.  *Lizzybel* realized that a solution, in which a mixed-in `token` would just re-dispatch to the original `token`, would be the best.  But alas, after about two days of hacking, it turned out to be still impossible do so in a transparent manner.

So the next best thing was to integrate the `Slang::Emoji` functionality into `Slang::Nogil`: an emoji could be considered to be a sigilless identifier after all, could it not?

The result was [Slang::Nogil 1.2](https://raku.land/zef:lizmat/Slang::Nogil/changes?v=1.2).

## Not enough testing: more trouble

*Lizzybel* had only tested the most simple cases.  But not something like `my Int answer = 42`.  Which fails with a "Two terms in a row" error.  Or something even worse, that would affect a lot of code in the wild: `my sub a() { }`, which **also** would fail in the same way.

Clearly the "sigilless-variable" approach would either require a more general approach in the Raku grammar, or would involve some serious ad-hoc workaround hacking in the `Slang::Nogil` modue.

Because it was nearly Xmas, **Lizzybel** opted for the ad-hoc workaround hacking approach for now.  "At least people would be able to play around with the use of sigilless variables in Raku, which some people would consider a nice Xmas present" was *Lizzybel*'s line of thought.

And after some [hacking](https://github.com/lizmat/Slang-Nogil/commit/5eddcd4f5ba89a23efc89929701d9ffad33be59a#diff-44f047142f328d6f26a7d8d328c36f65423b817b3a2e29b4b8d38b9953352a57), [Slang::Nogil 1.3](https://raku.land/zef:lizmat/Slang::Nogil/changes?v=1.3) saw the light of day.

## Not always available, or?

*Nanunanu* found out about the latest update of `Slang::Nogil` and enthusiastically send a private message to *Lizzybel* on IRC: "Very nice, I always wanted to be able to not have to use sigils for variables with limited scopes.  And now I can!  But I would still always would need to load the `Slang::Nogil` module in my code, no?".

*Lizzybel* answered: "Yes, at the moment you would have to.  But fortunately, you can automate that as well with the [`RAKUDO_OPT`](https://docs.raku.org/programs/03-environment-variables#index-entry-RAKUDO__OPT) environment variable.  Just put `RAKUDO_OPT=-MSlang::Nogil` in your environment, and you don't need to think about it anymore!".  It was silent on the other end.  But that was just because *Nanunanu* was also busy with something else.

After a few minutes *Nanunanu* answered: "That's pretty cool, didn't know you could do that  :-).  But of course it would be nicer still if it was just part of Raku, wouldn't it?".

## A good question

"Should this be part of Raku, perhaps in the next language level?", wondered *Lizzybel*.  "And should this only apply to variable definitions?  Or also to signatures, so you would be able to do something like `for <a b c d> -> letter { say letter }`.  Or would that affect error reporting on common errors too much?  Or would we be able to change the grammar and error reporting in such a way that sigilless identifiers in signatures would not be a problem after all?".

"Perhaps it is time for a [language problem solving issue](https://github.com/Raku/problem-solving/issues/new?template=issue-template-language.md).  And an associated [Rakudo Pull Request](https://github.com/rakudo/rakudo/pulls)", thought *Lizzybel*.  "But not now, as I'm still recharging my batteries".
