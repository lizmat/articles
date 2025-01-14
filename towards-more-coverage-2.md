# Finding the coverables

> This is part 2 in the ["Towards more coverage"](https://dev.to/lizmat/towards-more-coverage-fne) blog series.

The first blog described how to get a (big) file with all of the source locations that a particular Raku program executed.  But that log does **not** tell yet which lines *could* be executed, but *weren't*.  This blog post describes the development process of the logic to find out which lines of source-code *can* be executed, so that it's possible to report which lines were *missed*.

## Code::Coverable

I decided to put all the logic for finding lines in a separate distribution and called that [`Code::Coverable`](https://raku.land/zef:lizmat/Code::Coverable).  My main reason was that this logic should in principle be disconnected from the coverage logic, to allow for separate updates.  And maybe someone would find a use for this information in non-coverage context.

My initial thought was that RakuAST was mature enough to be used for such an endeavour, because when the Raku grammar parses a piece of source code, it remembers where all parts in the resulting AST originally came from in the source code.  But sadly that approach turned out to be not ready for prime-time just yet: if only a single thing would need to not be supported yet by RakuAST, all of the logic would fail.

So another approach was needed.  Then I realized that the MoarVM bytecode file **knows** which lines of source code can be executed, because that's where it gets the information from to produce the coverage log.

In April 2024 I had already spent significant time in understanding the MoarVM bytecode file structure, and made a [`MoarVM::Bytecode`](https://raku.land/zef:lizmat/MoarVM::Bytecode) distribution to solidify that understanding.  This makes the MoarVM bytecode a lot easier to inspect and to spot any anomalies.

> It comes with a number of helper scripts: but more about that in another blog post!

After some spelunking, I added a method [`coverables`](https://raku.land/zef:lizmat/MoarVM::Bytecode#coverables) to the `MoarVM::Bytecode` class that would return the line numbers of source code that *could* be executed.

## A little complication: what?
And then there's always that little bit extra complication.  One might think that a source-file can only contain lines of source code of that file.  And technically, that's true.  However, there's a little known feature in Rakudo: the `#line` directive.

> Actually, not sure if this is supposed to be a Raku Programming Language feature or not.  Assuming for now it is not.

The `#line` directive takes the form of `#line 42` (aka, just a line number), or `#line 42 a-different-file-name.rakumod` (a line number and something to indicate the (original) source-file.  It is typically used by all system programs that concatenate multiple source-files into a single source-file for compilation.

This allows one to fool the Raku parser to assume a different line number in the source code.  This in itself *may* be useful.  But more importantly, it allows one to specify a different file name, e.g. in the case of generated code, or if multiple source-files are concatenated (for whatever reason).

> An example of this can be found in the `gen/moar` directory of your Rakudo installation: the `CORE.c.setting` file: at this moment, it is the concatenation of 248 source files, and has reached a size of about 3.2MB with more than 98K LoC.  Clearly, one doesn't want to be debugging *that*, but rather the sources that created this enormous source-file.

Coming back to the `MoarVM::Bytecode.coverables` method, this also meant that it could not just return line numbers, but would have to return the line numbers keyed to whatever was known about the source-file at that point.  Which is why `MoarVM::Bytecode.coverables` returns a `Pair`, with the key as what was specified with any `#line` directive (which, as you may expect, defaults to the actual source file if no `#line` directives are specified), and the line numbers as the value.

This key will be known as the "coverage key" from now on.  And the `MoarVM::Bytecode.coverables` method may return more than one of these `Pair`s.

## Another complication: where?
Ok, so now it is possible to obtain the coverables from a `MoarVM` bytecode file.  But where do these bytecode files actually live?  How can one find the bytecode file for a `use` target like "String::Utils"?

Well, this information is kept in the internals of the `CompUnit::Repository::` modules.  These internals are pretty fiddly, so I decided to provide a somewhat easier interface for that information in the [`Identity::Utils`](https://raku.land/zef:lizmat/Identity::Utils) distribution, specifically the [`bytecode-io`](https://raku.land/zef:lizmat/Identity::Utils#bytecode-io) subroutine.

> Finding out how to do this was also an interesting journey, maybe worthy of another blog post at another time.

So now we can test this with e.g. `String::Utils`, one of the dependencies of the `Identity::Utils` module (so we can be sure that it is installed):
```raku
use Identity::Utils <bytecode-io>;  # only need "bytecode-io"
use MoarVM::Bytecode;

with bytecode-io("String::Utils") andthen MoarVM::Bytecode.new($_) -> $M {
    .say for $M.coverables;
}
```
which produces something like:
```
site#sources/2C5249974A64FCC88F16184290DB8B43D6DBE5FA (String::Utils) => [1 6 8 
25 26 46 47 66 67 72 73 78 79 80 81 84 85 102 104 105 109 110 111 112 115 116 135
137 138 142 143 144 149 150 151 152 153 154 155 165 168 169 182 183 184 185 186
187 188 189 190 191 192 193 194 195 196 198 200 201 203 204 208 209 211 216 217
218 219 220 221 222 228 233 235 236 238 241 242 247 248 258 259 260 261 262 271
282 285 286 287 290 291 296 299 300 304 305 310 ...]
```
Note that the `site#sources/2C5249974A64FCC88F16184290DB8B43D6DBE5FA (String::Utils)` key in there is is the "coverage key".  It doesn't look like a path, and indeed it isn't.  It's an abstraction that the `CompUnit::Repository::` modules use to find the (static version of the) source-file that was used to create the bytecode file.

> When modules are installed, they are typically moved to a secure location to ensure that they are not changed.  The coverage key allows the `CompUnit::Repository::` module that is in charge to locate the source file when necessary (e.g. for a re-precompilation if the version of Rakudo changes).

## Final complication: when?
The coverables information in bytecode files is determined by the lines of source seen by the grammar.  However, after the initial AST (Abstract Syntax Tree) is made, the static optimizer starts making changes to the tree to get faster and more memory efficient bytecode.

One of the most obvious results of this, is that `else` statements are *almost never* covered during execution, even if the contents of the body of the `else` statement *is* clearly covered.  So this causes a lot of false negatives in a coverage report.

> My initial idea was to look at the actual bytecode to see if anything could be deduced from that.  A lot can be deduced from the actual bytecode, but **not** whether a specific line of source-code is coverable.  So I quickly skipped that idea.

Also, anything that is executed at compile time, will **never** be covered by runtime (because it is already executed before runtime, to set up all that is needed during runtime, and does not need to be executed again)..  So the alternative is to look at the source code and develop some heuristics to recognize lines that will most likely not be covered:
- a line starting with `else`
- lines starting with `constant`, `enum`, `subset` or `proto`
- lines starting with `BEGIN`
- various other heuristics

> Of course, some lines of source-code will be marked as "uncoverable" even if they aren't.  That's ok, as we're really only interested in lines that are "coverable" but **not** covered.

In the end, I found a few cases in my extensive set of module distributions that proved to be hard to create heuristics for, e.g. lines in an extensive `BEGIN` block (which will only be executed at compile time).  For those cases I added the heuristic: "line ends with **# UNCOVERABLE**".  This allows module developers to mark these lines preventing them from negatively affecting coverage reports.

## Wrap it up
Finally, I decided to wrap it all up in a [`coverables`](https://raku.land/zef:lizmat/Code::Coverable#coverables) subroutine that produces `Code::Coverable` objects, which have these attributes:
- target - the `use` target specified, e.g. "String::Utils"
- key - the "coverage key" obtained from the bytecode
- line-numbers - a `List` of unique line-numbers, in ascending order
- source - an `IO::Path` to the associated source-file

## Conclusion
This was by far the most difficult part of getting test coverage for Raku modules into a viable product.  It involved a lot of yak shaving and detours, but in the end created a nice set of capabilities in the [`Identity::Utils`](https://raku.land/zef:lizmat/Identity::Utils) and [`MoarVM::Bytecode`](https://raku.land/zef:lizmat/MoarVM::Bytecode) modules.  And the new [`Code::Coverable`](https://raku.land/zef:lizmat/Code::Coverable) module, of course :-).
