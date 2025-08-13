# CRA and Open Source

> This is part 2 in the [SBOM](https://dev.to/lizmat/series/32933) series of blog posts

To start with giving credit where credit is due: *Salve J. Nilsen* has been instrumental in making me aware of the oncoming [Cyber Resilience Act](https://en.wikipedia.org/wiki/Cyber_Resilience_Act) effects.  Many presentations on this subject at various open source events have been given by him in the past years, and the videos and the slides and the chats have helped me a lot in trying to get a grip on the subject matter.  A large part of this blog post has been derived from those presentations.  I hope we'll be able to work together more on this in the future!

## Some aspects of the CRA

To give an idea of the timescale of the CRA:
- Into effect: *10 December 2024*
- Severe issue reporting obligation: *11 June 2026*
- Other reporting obligations: *11 September 2026*
- Main obligations: **11 December 2027**

So yes, the law is actually in effect already!

The [slides of a presentation](https://security.metacpan.org/presentations/gpw2025-cpan-security-sustainability/#/2) that Salve gave in Germany earlier this year are very enlighting.  Some of the points in that presentation:

- Supplying incorrect, incomplete, and/or misleading information may be fined up to 5M EUR or 1% of global turnover
- The CRA applies to all products with digital elements, including:
  - Connected devices
  - Non-tangible digital products
  - […their] remote data processing solutions
  - […and] related systems and services needed for operation
- Produce proof upon request by regulators certifying no known vulnerabilities

Which means businesses will soon need to take steps to become compliant in time for the deadline!

## Does the CRA directly affect open source developers

No.  Well, almost no.  As a person working on, or contributing to an Open Source project, you do not *have* to worry about the CRA, as long as any of these statements are true:
- you contribute code to projects you are not responsible for
- are **not** monetising the product
- make a product that is ultimately not intended for commercial activities

Yes, that last one is vague.  And probably intentionally so: the future will tell what that means exactly in practice.

But the goal of the CRA is clear: to increase the level of security of software in general.  Which is a *really* good goal.  So while it may not affect maintainers directly, it **will** affect maintainers indirectly.

The option developers and maintainers of Raku module distributions ought to consider, is what kind of information they can provide to make it easier for manufacturers to become compliant - eg. by enriching the [META information](https://github.com/Raku/problem-solving/issues/491) of a project.

A maintainer will need to decide whether under what conditions they would want to take part in the security regime that the CRA envisions (such as promises to respond in a timely manner to security issues).  That *could* be a "no" (the module was presented as is).  Or a "no, unless..." (fixes provided can be applied and/or developed with some kind of support).

## Open Source Steward

The CRA introduced a new concept of an organization in the world: the [Open Source Steward](https://www.developer-tech.com/news/open-source-wins-concessions-new-eu-cyber-law/).  An Open Source Steward is a legal entity that:
- provides sustained support for Open Source products that are ultimately intended for commercial activities
- plays a main role in ensuring viability of these
- faciliates due diligence obligations and cooperates with market surveillance authorities
- notifies on security incidents and vulnerabilities

Should all of these functions be done by volunteers?  Possibly, but that would just decrease the amount of fun one can have in Open Source projects.  And why should they?  If an Open Source component is used by a company for profit, that company should give back to the community in one shape or form.  In the case of open source communities such as the Raku community, this has been done by donations and sponsoring some organizational body.  Which in turn would give grants to developers and maintainers.

> A small rant: the disadvantage of this approach is that it is voluntary.  It's fundraising from a position of weakness, rather than strength.

But with the CRA this changes: rather than giving to Open Source communities basically for products already delivered (and the hope for continuation), companies will now need timely and accurate replies to their assertion queries.

A properly financed Open Source Steward could play a role in this.  Such a financially healthy Open Source Steward would then be able to facilitate further development of projects through financial and organisational means.  Which might make it possible for some Open Source developers to reach their dream: working on Open Source **and** be able to make a living.

## Conclusion

The CRA should become a reason for companies to use **more** Open Source, not less!  If we as Open Source enthusiasts can make it easier (and cheaper) for companies to use Open Source, while at the same time making Open Source more sustainable through Open Source Stewards being supported by companies, we have created a win-win situation for all involved.
