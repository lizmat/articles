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


Quite a lot of YAML in there.  But the gist is basically clear: run the tests of this module on the latest Ubuntu / MacOS / Windows operating systems, and use the latest Raku version for that.  It was really great to see how easy it was to automatically get Continuous Integration support for your module.

And of cours, as a module developer, you will get a notice (an email in this case) if testing of a module would fail.  That's how yours truly found out that many modules that rely `NativeCall` calls to C-library functions that depend on POSIX semantics, will simply fail on Windows.
