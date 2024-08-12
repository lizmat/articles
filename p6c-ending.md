# The end of p6c

In the past 2 years, module distributions in the Raku Programming Language that were being published through the original (but deprecated) "p6c" ecosystem, were no longer being harvested by the [default "zef" harvester](https://raku.land/github:ugexe/App::ecogen).  But they were still being harvested by the Raku Ecosystem Archive harvester.  But this also stopped with the [0.0.26 release of Ecosystem::Archive::Update](https://raku.land/zef:lizmat/Ecosystem::Archive::Update/changes?v=0.0.26).  And thus, any updates to these distributions remained visible.

> The reasoning for no longer supporting the "p6c" ecosystem is explained in the problem solving issue ["Preparing the Raku Ecosystem for the Future"](https://github.com/Raku/problem-solving/issues/316).

This means that any updates to the *651* distributions still in the "p6c" ecosystem, will *not* be noticed anymore in the live Raku ecosystem.  To alert the authors / current maintainers of these distributions, *202* issues were generated, listing the distributions affected.

Now, one week later, many authors responded to the issue they found in one of their repositories.  The reactios were generally positive to this effort.  Some of the authors took this notice as an opportunity to update their distribution to the "fez" ecosystem.  Kudos to these authors:

- YellowApple for [GLFW](https://raku.land/zef:YellowApple/GLFW)
- Itsuki Toyota for:
  - [Algorithm::BinaryIndexedTree](https://raku.land/zef:titsuki/Algorithm::BinaryIndexedTree)
  - [Algorithm::KdTree](https://raku.land/zef:titsuki/Algorithm::KdTree)
