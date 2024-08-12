# The End Of p6c

In the past 2 years, module distributions in the [https://raku.org](Raku Programming Language) that were being published through the original (but deprecated) "p6c" ecosystem, were no longer being harvested by the [default "zef" harvester](https://raku.land/github:ugexe/App::ecogen).  But they were still being harvested by the Raku Ecosystem Archive harvester.  And thus, any updates to these distributions remained visible.

But this harvesting stopped with the [0.0.26 release of Ecosystem::Archive::Update](https://raku.land/zef:lizmat/Ecosystem::Archive::Update/changes?v=0.0.26).

> The reasoning for no longer supporting the "p6c" ecosystem is explained in the problem solving issue ["Preparing the Raku Ecosystem for the Future"](https://github.com/Raku/problem-solving/issues/316).

This means that any updates to the *651* distributions still in the "p6c" ecosystem, will *not* be noticed anymore in the live Raku ecosystem.  To alert the authors / current maintainers of these distributions, *202* issues were generated, listing the distributions affected.

Now, one week later, many authors responded to the issue they found in one of their repositories.  The reactios were generally positive to this effort.  Some of the authors took this notice as an opportunity to update their distribution to the "fez" ecosystem.  Kudos to these authors:

- YellowApple for [GLFW](https://raku.land/zef:YellowApple/GLFW)
- Itsuki Toyota for:
  - [Algorithm::BinaryIndexedTree](https://raku.land/zef:titsuki/Algorithm::BinaryIndexedTree)
  - [Algorithm::KdTree](https://raku.land/zef:titsuki/Algorithm::KdTree)
  - [Algorithm::Kruskal](https://raku.land/zef:titsuki/Algorithm::Kruskal)
  - [Algorithm::TernarySearchTree](https://raku.land/zef:titsuki/Algorithm::TernarySearchTree)
  - [Algorithm::Treap](https://raku.land/zef:titsuki/Algorithm::Treap)
  - [Algorithm::ZobristHashing](https://raku.land/zef:titsuki/Algorithm::ZobristHashing)
- Fernando Correa de Oliveira for:
  - [SixPM](https://raku.land/zef:FCO/SixPM)
  - [Heap](https://raku.land/zef:FCO/Heap)
  - [Injector](https://raku.land/zef:FCO/Injector)
  - [Protocol](https://raku.land/zef:FCO/Protocol)
  - [Trie](https://raku.land/zef:FCO/Trie)
- Brian Duggan for [Slang::Mosdef](https://raku.land/zef:bduggan/Slang::Mosdef)
- Haytham Elganiny for:
  - [Grid](https://raku.land/zef:hythm/Grid)
  - [Retry](https://raku.land/zef:hythm/Retry)
- Dean Powell for [Locale::Codes::Country](https://raku.land/zef:PowellDean/Locale::Codes::Country)

Other authors responded that they don't want to spend any time on these distributions anymore, but would like to have them transferred to the [Raku Community Module Adoption Center](https://github.com/raku-community-modules/).  Kudos to these authors for spending their time and effort on these distributions so far and making them available for the future users of the Raku Programming Language:

- Yechen Fu for [Redis](https://raku.land/zef:raku-community-modules/Redis)
- brian d foy for:
  - [Chemistry::Elements](https://raku.land/zef:raku-community-modules/Chemistry::Elements)
  - [PrettyDump](https://github.com/raku-community-modules/PrettyDump) (in progress)
- Yann Büchau for: [Fortran::Grammar](https://raku.land/zef:raku-community-modules/Fortran::Grammar)
- +merlan #flirora for:
  - [Math::Random](https://raku.land/zef:raku-community-modules/Math::Random)
  - [Terminal::WCWidth](https://raku.land/zef:raku-community-modules/Terminal::WCWidth)
- Carsten Hartenfels for: [Text::Markdown::Discount](https ://github.com/raku-community-modules/Text-Markdown-Discount) (in progress)
- loren for:
  - [Net::FTP](https://github.com/raku-community-modules/Net-FTP) (in progress)
  - [Net::FTPlib](https://github.com/raku-community-modules/Net-FTPlib) (in progress)
  - [App::snippet](https://github.com/raku-community-modules/App-snippet) (in progress)
- Rob Hoelz for:
  - [Algorithm::Elo](https://raku.land/zef:raku-community-modules/Algorithm::Elo)
  - [Algorithm::LCS](https://raku.land/zef:raku-community-modules/Algorithm::LCS)
  - [Pod::EOD](https://raku.land/zef:raku-community-modules/Pod::EOD)

Note that the distributions marked "in progress" still need some Tender Loving Care before they will be properly integrated into the Raku ecosystem again.  Pull Requests for these distributions are *very* welcome!

A number of other authors responded that they thought that (some of) their distributions were not fit to be modernized or transferred to the Raku Module Adoption Center.  Kudos to these authors nonetheless, because of their time and efforts in the past.  Even if that didn't result into something they thought was worth salvaging.  These distributions have been removed from the "p6c" ecosystem, which thusly contains *552* distributions now: a 15% reduction in one week!

> Yours truly will actually look at all of these packages to see whether they warrant a transfer to the Raku Module Adoption Center.  Beauty **is** in the eye of the beholder  :-)

All in all a fruitful week in the Raku Ecosystem world.

## And now?

Many authors that received an issue notice about this, have not responded yet.  If you are one of them, please respond to the issue and tell us what you would like (us) to do.

Meanwhile the distributions marked "in progress" could use someone looking at the code and/or the tests to see what's stopping inclusion into the Raku ecosystem again.  Many kudos in advance.

And if you're an author who already transferred their modules to the "zef" ecosystem: thank you for you continued practical support of the Raku Programming Language, and the extension of the [https://raku.land](Raku Ecosystem)!
