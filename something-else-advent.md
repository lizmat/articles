# Something else

Santa was absent-mindedly going through the Rakudo commits of the past weeks, after hearing about the new [2022.12 release](https://rakudo.org/post/announce-rakudo-release-2022.12) of the Rakudo compiler.  And noticed that there were no commits after that release anymore.  Had all the elves been too busy doing other stuff in the Holiday Season, he wondered.  But, in other years, the Raku core elves had always been very busy in December.  He recalled December 2015 with a bit of a smile on his face: my, my, had the elves been busy then!

A little worried, he asked Lizzybel to come in again.  "So, why is nobody working on Rakudo anymore", he asked.  "Ah, that!", Lizzybel said.  "Not to worry, we changed the default branch of Rakudo to 'main', she said.  "Why would you do that?, Santa asked, showing a bit of grumpiness.  "Was the old default branch not good enough?".  Lizzybel feared a bit of a long discussion (again), and said: "It's the new default on Github, so us Raku core elves thought it would be a good idea to follow that, as many tools now assume 'main' as the default branch".

"Hmmrph", said Santa, while he switched to the 'main' branch'.  "Wow!, more than 780 commits since the 2022.12 release, how is that possible?", he exclamed.  "Don't the elves have nothing better to do in this time of the year?" he said, while raising his voice a bit.  Lizzybel noticed his cheeks turning a little redder than usual.

"Ah that!", said Lizzybel again.

# RakuAST

And she continued, again.  Remember the RakuAST project that was started by the main MoarVM elf about two and a half year ago?  It's been in off-and-on development since then, and now the core elves deemed it ready enough to make the work so far, available in the 'main' branch.  So that other core and non-core elves could easily try out some of the new features that it is providing.  "So, it's now done, this RakuAST project?", said Santa, with a little glimmer of hope in his eyes.  "Ah, no, you could say that the project is now more than halfway", Lizzybel said, hoping it would be enough for Santa.  "Remind me again what that project was all about?", Santa said, destroying Lizzybel's hope for a quit exit.

While sitting down, Lizzybel said: "An AST can be thought of as a [document object model](https://en.wikipedia.org/wiki/Document_Object_Model) for a programming language. The goal of RakuAST is to provide an AST that is part of the Raku language specification, and thus can be relied upon by the language user. Such an AST is a prerequisite for a useful implementation of macros that actually solve practical problems, but also offers further powerful opportunities for the module developer.  RakuAST will also become the initial internal representation of Raku programs used by Rakudo itself. That in turn gives an opportunity to improve the compiler."  "I bet you had ChatGPT type that out for you to memorize", Santa said with a twinkle in his eye.

"Eh, no, actually, this is from the [MoarVM's elf grant proposal in 2020](https://news.perlfoundation.org/post/gp_rakuast), confessed Lizzybel.  "Ok, so tell me what are the deliverables of that project?  I don't have all day to look through grant proposals, you know", said Santa.

Lizzybel peeked at her elfpad, took a deep breath and said: "Well, firstly: class and role implementations defining an document object model for the Raku language and its sub-languages, constructable and introspectable from within the Raku language.  Secondly, the generation of QAST, the backend-independent intermediate representation, from RakuAST nodes, such that one can execute an AST.  Thirdly, tests that cover the running of RakuAST nodes.  And lastly, integration of RakuAST into the compilation process."  "Interesting", said Santa, "and how much of that is done already?".  "Enough to make more than 60% of the Rakudo test files pass completely, and more than 40% of the Raku test files pass completely", said Lizzybel.

# Use it now

Santa continued what was now feeling like an interrogation.  "So what use is RakuAST now?"  "Well, it allows module developers to start playing with RakuAST features", said Lizzybel.  "But are you sure that RakuAST is stable enough for module developers to depend upon?", Santa said, frowning.  "No, the core elves are not sure enough about it yet, so that is why module developers will need to add a `use experimental :rakuast` to their code."  "Is there any documentation of these RakUAST classes?"  "No, not really, but there are test files in the [t/12-rakuast subdirectory](https://github.com/rakudo/rakudo/tree/main/t/12-rakuast).  And there is a proof-of-concept of a module that converts [`sprintf` format strings](https://docs.raku.org/routine/sprintf#Directives) into executable code in the new [`Formatter`](https://github.com/rakudo/rakudo/blob/main/src/core.e/Formatter.pm6) class", Lizzybel blurted out.

"Ok, that's a start", said Santa, with a lighter shade of red on his cheeks.

Then Santa was distracted by the snow outside again and mumbled: "Are the reindeer prepared now?".
