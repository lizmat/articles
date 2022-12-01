# Something old, something borrowed, something new, something stashed

Santa, having a little time off earlier this year, was looking at [all of the modules](https://raku.land) that the Raku elves had made over the years, now over 2000 of them!  But then he noticed something: not all of the modules appear to come from the same ecosystem?  So what's going on here, he asked one of the Raku core elves, Lizzybel.  "It's complicated", she said.  And continued:

## Something old

You see, a long time ago, when Raku was but an alpha-version programming language, some of the Raku elves started writing some useful modules.  But it was problematic to distribute and install them.  You would need to know the exact URL of the source of the module to be able to download, and install it.

So some smart elf realized that if there would be a single list of those URLs on the interwebs, it would be possible to read that regularly, see if there are any new or changed entries, and create a small database (well, actually a JSON file) from introspecting all of the information in those modules.  With that database, you could ask for a module name, and it would give you the URL where the code was actually located.  Writing a script that would download and install a module given a module name was relatively easy after that.

The main problem with this approach is, that if an elf would update a module *without* updating the version information, people could get different versions of a module, even though they had the same version number.  And from a security point of view, nobody was checking whether the uploader actually matched the "owner" of the module.  And although no impersonations are known to have happened, it is definitely not something the Raku elves want to continue to support in the long run.

"So, this is the original Raku ecosystem?", Santa asked.  "Yes, indeed", Lizzybel said, "and us Raku core elves sometimes refer to this as the 'p6c' ecosystem, for various hysterical raisins".

"Very drole", Santa mumbled.

## Something borrowed

"But what about that CPAN ecosystem I saw on [raku.land](https://raku.land/stats)?", said Santa.  "Ah that, eh", said Lizzybel.  And continued again:

When it became clear that the first official release of Raku was going to happen, I asked at a Toolchain Summit with the Perl elves, whether we should try to get a shared module ecosystem or not.  The majority of the elves thought it would be a good idea to pool resources in that respect.  And so Perl and Raku elves worked a lot on making the underlying storage system of the Perl ecosystem (aka CPAN) handle Raku modules as well.  Now you only needed to get a PAUSE login (the upload system of CPAN), and mark your module as being a Raku module, and you would be set!

The CPAN system had the advantage that we would at least be sure who had uploaded a module.  But it doesn't check whether it matches the internal information of the module.  So it still has the potential for abuse.

"Yeah, that's still not ideal", said Santa.

## Something new

"Indeed not", said Lizzybel.  And once more continued:

So some other smart elves decided it was time to get a proper Raku solution for the ecosystem.  A place where not only we would know who had uploaded a module, but also a place where the internal information of a module was checked to see if it matched with the uploader.  They were basically the same elves that had made the new module installer ["zef"](https://raku.land/github:ugexe/zef), and they thought it would be appropriate to call the new module upload logic ["fez"](https://raku.land/zef:tony-o/fez) ("zef" for download, "fez" for upload).

This ecosystem has all of the features we want for the future of Raku.  Too bad the majority of the modules is still in the other ecosystems.

"So, the Raku elves should be moving to the "zef" ecosystem?", wondered Santa.  "Yeah, that would be best", said Lizzybel, hoping that Santa wouldn't know about [the sunsetting announcement](https://github.com/Raku/problem-solving/blob/master/solutions/meta/sunsetting-p6c-cpan.md#sunsetting-p6c--cpan-ecosystems).  "Ah, now I remember something about these older ecosystems, weren't they supposed to be phased out earlier this year?", said Santa without raising his voice much.  Lizzybel blushed, and said: "Yeah, it was supposed to.  But so much has happened in 2022, it was hard to not be distracted".  Santa nodded and said "Indeed it was, and it still is" with a look of understanding and sadness.

## Something stashed

Then Santa showed that there was a bit of a devious streak in him: "hmmm... so what would happen if a naughty elf would remove a module from the ecosystem?  Wouldn't that potentially cause problems for other elves using that module in production?".  Lizzybel glowed a bit: "Yes, it would.  Because of that, I implemented the [Raku Ecosystem Archive](https://github.com/raku/REA#raku-programming-language-ecosystem-archive).  It contains all versions of all Raku modules that ever existed.  Well, that were still available when I started the Raku Ecosystem Archive harvester, about a year ago.  And the Raku elves who made "zef" made it fallback to that, so that you should be able to install any module forever".  "Aha", said Santa, "so do elves that upload modules need to do anything special to have their module archived?".  "Nope", said Lizzybel.  "Nice", said Santa.

Then Santa was distracted by the snow outside and mumbled: "Better get the reindeer prepared".
