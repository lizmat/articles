# Creating coverage information

> This is part 3 in the ["Towards more coverage"](https://dev.to/lizmat/series/30086) blog series.

The first blog introduced the new [`Test::Coverage`](https://raku.land/zef:lizmat/Test::Coverage) module that allows a developer to test how well the tests of a distribution actually check the code of a distribtion.  The second blog delved into the development process of making the [`Code::Coverable`](https://raku.land/zef:lizmat/Code::Coverable) distribution that provides the logic to the determine the lines in a source-file that *can* be covered.

## Code::Coverage
With the `Code::Coverable` module now being operational, it was time to package this up in a module that would actually create coverage information.

And that became the [`Code::Coverage`](https://raku.land/zef:lizmat/Code::Coverage) module.  And in its simplest use, it looks like this:
```raku
use Code::Coverage;

my $coverage = Code::Coverage.new(
  targets => @targets,
  runners => @test-scripts
);

$coverage.run;

say .key ~ ": " ~ .value for $coverage.missed;
```
At object instantiation, provide two named arguments.

The first one is called `targets` and should contain the `use` targets for which to provide coverage information.  These use targets are usually the `key` of the `Code::Coverable` object returned by the `coverables` subroutine of the `Code::Coverable` distribution.

> In the `Test::Coverage` case, this would be obtained from the "provides" section from the `META6.json` file of a distribution.

The second one is called `runners`, and it should contain the path(s) of the scripts that should be executed to create coverage information.

> In the `Test::Coverage` case that would be set up with all the `.t` and `.rakutest` files in the `t` and `xt` directories of the distribution.

Then, call the `run` method on the `Code::Coverage` object to execute the scripts and create the coverage information, scan and process it.

After that, you can call one of several report methods, such as `missed`, which will then report the line numbers of the code that was **not** covered.

## Multiple runs

Sometimes you may want to run the same script multiple times, but with different arguments, to get complete coverage information.  And you can!  Just call the `run` method once again, and specify any command line arguments as arguments to `run` method:
```raku
$coverage.run("foo");
$coverage.run("bar");
```
After each call, the coverage information is updated and you can call any of the report methods again.

If the different code paths depend on environment variables, you should just set those in the `%*ENV` variable:
```raku
%*ENV<FROBNICATE>=1
$coverage.run;  # run with frobnication
```

## Annotations

It is always nice to know the line numbers of code that didn't get covered.  But that would still be a lot of looking up and down the code to find out what the contents was of the lines that didn't get covered.

To make that process easier, the `Code::Coverage` object has an `annotated` method that will produce the source code of a `use` target, and prefix each line of source with one of these four possibilities:

- "`* `" - line was coverable, and covered
- "`✱ `" - line was **not** coverable, but covered anyway
- "`x `" - line was coverable and **not** covered
- "`  `" - line was not coverable

Looks familiar?  It should be if you've read the first post of this blog series!  Yes, that's indeed the logic that `Test::Coverage` uses to create the `.rakucov` files.

## More control

The [`new`](https://raku.land/zef:lizmat/Code::Coverage#new) method allows for more customization / configuration of the coverage process, such as an option to specify where (temporary) coverage files are to be stored, and whether they should be removed or not after processing.

It also allows you to inspect the output that the scripts produced in each call to the `run` method (both STDOUT and STDERR), if you would like to find out what the scripts actually did.

## Conclusion

As you may have gathered: the `Code::Coverable` and `Code::Coverage` modules are the workhorses of the `Test::Coverage` module, which is basically a thin layer around these modules.  I find that to be a good design principle: have a "backend" (computer) and a "frontend" (human interaction) functionality.  Or in "git" terms: separated in "plumbing" and "porcelain".

*If you like what I'm doing, committing to a [small sponsorship](https://github.com/sponsors/lizmat/) would mean a great deal to me!*
