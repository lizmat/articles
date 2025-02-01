# Test coverage in practice

> This is part 4 in the ["Towards more coverage"](https://dev.to/lizmat/series/30086) blog series.

The [first blog post](https://dev.to/lizmat/towards-more-coverage-fne) showed the basic usage of the [`Test::Coverage`](https://raku.land/zef:lizmat/Test::Coverage) distribution.  But it can do more!

## All code covered?

If you have a situation where all of the test in a distribution cover all of the executable code, you can simplify the test file to:
```raku
use Test::Coverage;

must-be-complete;
```
This will set the `plan` to 1 test, and mark it ok if indeed all code is covered.

Should that fail for some reason, then the [`report`](https://raku.land/zef:lizmat/Test::Coverage#report) and [`source-with-coverage`](https://raku.land/zef:lizmat/Test::Coverage#source-with-coverage) subroutines will be called, providing the developer with feedback on which lines were not covered.

If you want to be clear to yourself what the goal is for coverage testing, you could mark this subroutine as "todo":
```raku
use Test::Coverage;

todo "needs more tests!";
must-be-complete;
```
This will make sure that on the one hand the failure of coverage testing will **not** stop you from uploading a new release with e.g. [`App::Mi6`](https://raku.land/zef:skaji/App::Mi6), but will make it clear to any current or future maintainer that the intent was to cover all code in tests.

Personally, I wouldn't do this though.  Coverage testing feels too much like bondage to me then, and programming really should be fun!

## Are all code paths really covered?

Even if the tests say that all lines of code are covered, does that guarantee that all code paths are covered?  Probably not.  Why is that?

Well, that's because the granularity of code coverage is **source line** based.  And a lot can happen in one line of Raku source code!  For example, take this simple ternary:
```raku
my $a = <foo frobnicate>.pick;  # either "foo" or "frobnicate"
my $b = $a eq "foo" ?? "bar" !! "baz";
```
From a code coverage point of view, it doesn't matter whether `$a` had the value `"foo"` or `"frobnicate"` in the assignment to `$b`.  But for the execution of your code, that might really have different results!

One way around that, might be to split that ternary over multiple lines:
```raku
my $a = <foo frobnicate>.pick;  # either "foo" or "frobnicate"
my $b = $a eq "foo"
          ?? "bar"
          !! "baz"
;
```
But that also feels a bit icky: adapting your coding style just to satisfy any coverage testing.

It's probably better to realize that 100% coverage testing is still **no** guarantee that all possible code paths are tested.  And that you should always remain vigilant with regards to testing your code: find new ways to break it, and make test to spot such breakage!

## Status update

Of the [225 distributions](https://raku.land/zef:lizmat) that I am currently personally maintaining, 108 of them now have coverage testing enabled.  32 of them have complete coverage testing.  So that's still a lot of modules not being tested, and definitely a lot not fully covered.  After the initial push to test the coverage testing itself, I did about 100 of them.  That gave me a good idea of all of the corner cases.

Now I add coverage testing only if a distribution needs updating.  And any new distribution gets coverage testing, of course!

## What about RakuAST?

The new [`Slang::Lambda`](https://raku.land/zef:lizmat/Slang::Lambda) is a **very** small distribution that allows one to use `λ` as a synonym for the point block starter `->` in Raku source code.  This is [a *really* small module](https://github.com/lizmat/Slang-Lambda/blob/main/lib/Slang/Lambda.rakumod), so coverage testing should be easy, one would think.

The problem is that one piece of code is executed when the legacy grammar is used, and the other piece of code is executed if the new Raku grammar (RakuAST) is used.  And one can only be executing code in one or the other grammar!

Fortunately, the `Test::Coverage` distribution allows one to handle that case as well, by being a little more verbose.  The test file looks like:
```raku
use Test::Coverage;

default-coverage-setup;

run;

%*ENV<RAKUDO_RAKUAST>=1;
run;

must-be-complete;
```
Note that now we explicitely run the [`default-coverage-setup`](https://raku.land/zef:lizmat/Test::Coverage#default-coverage-setup) and [`run`](https://raku.land/zef:lizmat/Test::Coverage#run) subroutines (which would normally be run under the hood by the test routines).

Just before the second time the `run` subroutine is called, the `RAKUDO_RAKUAST` environment variable is set: this will cause the `raku` executable to use the new Raku grammar when parsing code.  In this case, it will mark the RakuAST specific code in the Lambda slang as executed.

The call to `must-be-complete` will then see that all is ok, and mark the test is ok.

## Conclusion

Coverage testing will help you determine which parts of the code in a distribution are not tested yet.  But is **no** guarantee that really all possible code path have been tested.

Adding coverage testing at a later stage, is a lot of work.  Make sure that you add it during development of a distribution, as you would with any other tests.

The `Test::Coverage` distribution allows for further flexibility in a lot of corner cases as well, not just for the simple, straightforward testing cases.

*If you like what I'm doing, committing to a [small sponsorship](https://github.com/sponsors/lizmat/) would mean a great deal to me!*
