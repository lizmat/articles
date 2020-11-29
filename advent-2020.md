Helping the Github Action elves
===============================

As a module developer, you are sometimes surprised by the tools that you use.  In this case, yours truly was surprised by a recent update of the excellent [App::Mi6](https://modules.raku.org/dist/App::Mi6:cpan:SKAJI) tool by *Shoichi Kaji*.  After an upgrade, it started adding a `.github/workflows/test.yml` file to new distributions.  And this in turn caused Github to test the distribution after each commit.  Which is great, especially if it finds problems!

So what did this file consist of?

```yaml
    name: test

    on:
      push:
        branches:
          - '*'
        tags-ignore:
          - '*'
      pull_request:

    jobs:
      raku:
        strategy:
          matrix:
            os:
              - ubuntu-latest
              - macOS-latest
              - windows-latest
            raku-version:
              - 'latest'
        runs-on: ${{ matrix.os }}
        steps:
          - uses: actions/checkout@v2
          - uses: Raku/setup-raku@v1
            with:
              raku-version: ${{ matrix.raku-version }}
          - name: Install Dependencies
            run: zef install --/test --test-depends --deps-only .
          - name: Install App::Prove6
            run: zef install --/test App::Prove6
          - name: Run Tests
            run: prove6 -l t
```

Quite a lot of YAML in there!  But the gist is basically clear: run the tests of this module on the latest Ubuntu / MacOS / Windows operating systems, and use the latest Raku version for that.  It was really great to see how easy it was to automatically get Continuous Integration support for your module.

And of course, as a module developer, you will get a notice (an email in this case) if testing of a module would fail.  That's how yours truly found out that many modules that rely `NativeCall` calls to C-library functions that depend on POSIX semantics, will simply fail on Windows.

Loving the green lights
-----------------------
It's always great to see green, like in the test results of [Hash::LRU](https://github.com/lizmat/Hash-LRU/actions/runs/337667222).  But something struck yours truly: more than **5 minutes** for testing?  Looking at the [timing of each step](https://github.com/lizmat/Hash-LRU/runs/1331938416?check_suite_focus=true) shows that the `Install App::Prove6` step took **over 3 minutes**!  While the actual tests of the module only ran for **two seconds**.  It looked like there was a lot of overhead involved, especially if a module did not have any external dependencies.

Now, when yours truly is testing out modules locally, this usually happens with:

```raku
    raku -Ilib t/01-basic.t  # or whatever test-file that shows a problem
```

Why?  Well, really because this allows yours truly to directly add any debugging code to the test-file in case of failures, to more easily track the bug down.  And if there's an execution error, `--ll-exception` usually gets added, like so:


```raku
    raku -Ilib --ll-exception t/01-basic.t  # make sure we get a *full* backtrace
```

However, if Continuous Integration testing comes up with an execution error, then you don't get a full backtrace usually, which often does not help tracing the issue.  Especially if you cannot reproduce the problem locally.

Making things faster, better and more economic
----------------------------------------------
So, why not embed this manual workflow in a nice script, and add that to the module.  And make sure that only that script get run?  That seems like an easy idea to implement.  And it was!  The script (called `run-tests`) basically became (slightly shortened for this blog post):

```raku
    my @failed;
    my $done = 0;
    for "t".IO.dir(:test(*.ends-with: '.t' | '.rakutest')).map(*.Str).sort {
        say "=== $_";
        my $proc = run "raku", "--ll-exception", "-Ilib", $_, :out, :merge;
        if $proc {
            $proc.out.slurp;
        }
        else {
            @failed.push($_);
            if $proc.out.slurp -> $output {
                say $output;
            }
            else {
                say "No output received, exit-code $proc.exitcode()";
            }
        }
        $done++;
    }

    if @failed {
        say "FAILED: {+@failed} of $done:";
        say "  $_" for @failed;
        exit +@failed;
    }

    say "\nALL {"$done " if $done > 1}OK";
```

And the final 6 lines of the YAML file were changed to

```yaml
      - name: Run Tests
        run: raku run-tests
```

Total timing of testing the [Hash::LRU](https://github.com/lizmat/Hash-LRU) module typically dropped to [below one minute](https://github.com/lizmat/Hash-LRU/actions/runs/375191322).  That's a lot of time and CPU cycles saved!  Of course, this will be less if there are dependencies that need to be installed.  But the shorter turn-around time, as well as seeing complete backtraces if something goes wrong, have definitely helped yours truly!
