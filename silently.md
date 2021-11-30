Silently
--------

Santa was working on some programs to handle all of the intracacies of modern-day just-in-time package delivering, and got annoyed by some parts of the program getting noisy because some elf had left some debug statements in there.  Ah, the joys of collaboration!

So Santa wondered whether there could be a way to be less distracted by what otherwise seemed to be a perfectly running program.  Looking at the [Wonderful Winter Raku Land](https://raku.land), after a little bit of searching, Santa found the [`silently`](https://raku.land/zef:lizmat/silently) module.  That was great!  It's a module that exports a single subroutine `silently` that takes a block to execute, and will capture all output made by the code running in that block.

Whereas Santa would first do:
````raku
assign-optimal-trajectory(@gifts);
````
and get a lot of unwanted output, now Santa could just do:
````raku
silently { assign-optimal-trajectory(@gifts) }
````
and get the same result without so much noise.

But alas, just before all gifts where on their way, it turned out that some gifts had somehow been lost, or at least not assigned a proper trajectory.  Now, Santa had the option of running the same program again, but *with* all of the noise.  And time was getting short!  But then Santa realized that *if* something had gone wrong, there would be an error message on `STDERR`.

And guess what, the [`silently`](https://raku.land/zef:lizmat/silently) module actually only **muffles** whatever noise was generated!  After running your code, you can still find out the noise it made on `STDERR`, because `silently` returns an object that you can call the `.err` method on to get all the text that was sent to `STDERR`.  So the code became:
````raku
my $muffled = silently { assign-optimal-trajectory(@gifts) }
if $muffled.err -> $errors {
    say $errors;
}
````
This allowed Santa to quickly find and fix the problem for the gifts that had gone wrong.  And Christmas was saved once again!

Later, some elves were reprimanded for leaving debug statements in production code.  They promised to not do it again.
