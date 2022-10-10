# It's time to rak! (Part 1)

A few months ago, I had a bit of a scare with a notebook showing signs of going nuclear (as in batteries growing up to about 3x their original size, dislodging the bottom plate).  In the end, all turned out well, thanks to [iFixit](https://ifixit.com), patience and a steady hand.

Not wanting to install Perl's `ack` utility on a clean temporary machine, made me write an alpha version of a Raku module [`App::Rak`, providing a similar utility: the `rak` CLI]((https://raku.land/zef:lizmat/App::Rak)).  Which I presented at the second Raku Conference: [Looking for clues with rak](https://conf.raku.org/talk/174).

Since then, the utility has seen two refactors: the first one was taking out the "plumbing" functionality into a [separate module](https://raku.land/zef:lizmat/rak).  The second one was rewriting the argument handling (now up to 135 options) to make it easier to produce better error messages, and to make it more maintainable.  And now it's at what I would like to think as "beta version" level.

## What is it?

So what is `rak` anyway?  Maybe the tag line explains it a bit:

> 21st century grep / find / ack / ag / rg on steroids

In other words, a utility to find stuff (usually text) from somewhere (usually a set of files on a filesystem) and present the results in some form.  BTW, kudos to *Damian Conway* for suggesting this tag line.

But aren't there quite a few utilities like that already?  Yes, there are.  But I think it being based on the [Raku Programming Language](https://raku.org), allows it to have some unique features that would be hard to implement in any other programming language.

The naysayers among you might want to point out that this utility will be slow, because it is programmmed in Raku.  Yes, it is *not* the fastest utility around.  The primary focus of this utility, is its whipuptitude (how easy it is to get something working from scratch) and manipulexity (the ability to manipulate complex structures).  Together with a user-friendly and customizable interface.  After that comes performance.  And since it *will* use whatever CPUs you have, you might be pleasantly surprised by the low wallclock time of a complex search action.

## Unique Features

So what are these unique features of the Raku Programming Language that would make *you* want to use `rak`?:

- searching based on NFG

In the Raku Programming Language, strings are internally encoded in "Normalization Form Grapheme".  That means that strings are interpreted in the way a person would interpret the visibly separate characters.  That means, for example, that at Raku string level there is *no* difference between **é** (E9 LATIN SMALL LETTER E WITH ACUTE) and **é** (65 LATIN SMALL LETTER E, 2CA MODIFIER LETTER ACUTE ACCENT).  It's always considered to be a *single* grapheme.  Same for something more complicated such as **🏳️‍🌈** (1F3F3 WAVING WHITE FLAG, FE0F VARIATION SELECTOR-16, 200D ZERO WIDTH JOINER, 1F308 RAINBOW), which is *also* a *single* grapheme in Raku.

- searching without caring for case or accents

All search programs allow one to search case-insensitively, usually referred to as "ignorecase".  The Raku Programming language has that feature as well, of course.  But it also has "ignoremark": this only looks at the base character, thus matching **é** with **e**, and vice-versa.  Just by specifying `--ignoremark` with your search!

- extended regular expressions

The Raku Programming Language has re-invented [regular expressions](https://docs.raku.org/language/regexes) to be more expressive and much more readable.  For instance, whitespace has no meaning inside Raku's regular expressions, so you can be liberal with whitespace to make the intent clearer.

- other reasons

Well, there were quite a few things I didn't like in similar utilities.  Some were just missing features (like calling an editor with the result of your search, search JSON files, search on git blame info, make mass changes to files, rename files according to an algorithm)).  I also didn't like the complete mix and mash of arguments and shortcuts and aliases.

## No short options, no aliases

That's why the `rak` utility comes *without* any short options: all options only come in a long name.  Now, this may look as a bold decision and one that feels like it is very user unfriendly.  However, if you're not using such a utility on a daily basis, you will quickly forget the correct incantation for what you were trying to achieve in a similar case a (not so) little while ago.  Whether they're long or small.

That is why `rak` comes with the capability of saving any combination of options, and give it a unique name.  Which can be shorter, or longer, whatever suits **you** best!  For instance, saving `--ignorecase` as `-i`, and `--ignoremark` as `-m`, would allow you to activate both in a search with `-im`.  Note that it doesn't need to be a single argument: it could be multiple!  And you can give such a save option a description, for yourself in the future, or for your co-workers.  And that could include the pattern to search for and/or the location to search.  So you can save a repeatable search query in a single shortcut.  And should you have forgotten your own shortcuts, there's a way to list all of the custom options that you have.

## What is needed?

If you are sufficiently teased by this introduction, and you want to try it out for yourself?  The first requirement, is having an implementation of the Raku Programming Language installed.  Rakudo is the most complete implementation of Raku currently.  You can [build from source](https://github.com/rakudo/rakudo) if you are so inclined, but the easiest way is to install one of the [Linux binary packages by *Claudio Ramirez*](https://nxadm.github.io/rakudo-pkg/), or to use any of the [other installation methods](https://rakudo.org/downloads).

Then you'll need [`zef`](https://github.com/ugexe/zef#readme), Raku's module installer.  It comes with most binary package installs.  With that available, all you need to do is to run `zef install App::Rak`.  And you're reading to `rak`!

## Some examples

````
# Search for literal string "foo" from the current directory
$ rak foo

# Search for "foo" in the "lib" directory
$ rak foo lib

# Search for "foo" only in files with .txt and .text extensions
$ rak foo --extensions=txt,text

# Find all files that have "lib" in their name from the current dir
$ rak lib --find

# Show all unique name fields in JSON files
$ rak --json-per-file '*<name>' --unique

# Show all lines with numbers between 1 and 65
$ rak '/ \d+ <?{ 1 <= $/.Int <= 65 }> /'

# Show number of files / lines authored by Scooby Doo in current repo
$ rak --blame-per-line '*.author eq "Scooby Doo"' --count-only
````

## Conclusion

This concludes part 1 of a series of blog posts I intend to write about `rak`.  Since this is now in beta, it is unlikely that features will change in a non-compatible way from now.

This is also my first blog post on dev.to.  I hope it will be the first of many to come.

If you have any comments, find bugs, have recommendations / ideas, please submit them as issues at the [App::Rak repository](https://github.com/lizmat/App-Rak/issues).  If you would like to have a more direct interaction, you can visit the [#raku-rak](https://web.libera.chat/?channel=#raku-rak) channel on [Libera.chat](https://libera.chat).

Thank you for reading all the way to the end!
