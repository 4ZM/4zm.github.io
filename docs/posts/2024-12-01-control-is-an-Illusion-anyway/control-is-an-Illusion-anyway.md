---
draft: false
date:
  created: 2024-12-02
slug: control-is-an-Illusion-anyway
hide:
  - toc
categories:
  - Code Review
  - Best Practices
---

# Control is an Illusion Anyway...

Code review is an indispensable practice in any self-respecting software engineering organization. We get higher quality code and knowledge sharing. It's awesome. But, it’s also *not* awesome. To be honest, it can be a nightmare. Sometimes we get blocked for days waiting for approval or end up in a never-ending context-switching game of review ping-pong.

<!-- more -->

Today, we can't imagine living without version control or unit tests. We know they have a bit of an overhead, but the alternative is much worse. Much, much worse. Your code would roll over and crush you before you could say "git rebase" without these practices.

After living with code reviews for many years, there is no way I'm ever giving that up either. It's just too valuable. But the pain felt is on a different order of magnitude compared to using git and writing unit tests. Getting blocked while waiting for a busy code owner can be incredibly frustrating. The only thing worse is getting pinged by impatient co-workers when you're in the zone, churning out code.

#### <p style="margin: 30pt">It doesn't have to be this way though, but there's a catch: the other way is terrifying. You will have to give up control.</p>

There's a way to make reviews rosy (or at least tolerable). But, before we get into all that, let's think about why we are doing code reviews in the first place.

## :watch: Eventually Consistent Quality

If I were to check out your main branch right now, what quality of code would I get? Are there bugs? Is there technical debt? Are there parts in need of refactoring? In anything but a toy project, the answer to these questions is yes. Quality is not binary—it's on a continuum.

Unit testing and code reviews will increase quality. If not, then you're definitely doing it wrong. The more well-written tests you have and the more people looking at the code, the more issues will be found. Remember Linus's law: "Given enough eyeballs, all bugs are shallow."

But software engineering is about making hard decisions. There are always tradeoffs. How many eyeballs do you need before shipping? Or phrased differently, how much quality is enough quality? When do we enter the realm of diminishing returns, where development velocity and other costs outweigh further investments in quality?

#### <p style="margin: 30pt">There are two major quality thresholds to consider: What is good enough to merge? and What is good enough to release?</p>

For some projects, you can do genuine continuous deployments. In this case, the two thresholds are considered the same. The trunk is always in a releasable state. If this is within reach for your project, you should pour libations in thanks to your favorite deity; this is paradise. For most projects, however, it's not practical to make them the same. But we always strive to keep the merge and release quality levels as close as possible.

Trunk quality should be high enough to not disrupt development. Code should compile, the unit tests should pass, and someone other than the PR author should have had a look at every change. I call it the "no regrets" level of quality: Developers should always be able to pull the main branch without regretting it.

The trunk quality should be high enough that making a release is a simple, predictable, and low-risk activity. Even so, you might need to tick a few more boxes to get it shipped. For instance, you might need to execute some long-running nightly tests. Or you might need to run tests on prototype hardware, a very limited resource that can't be horizontally scaled in CI. Or you might have tests that need to run on some very expensive system, like a GPU cloud cluster.

You should run these tests as often as you can afford to on your main branch. But running them on *all* PRs in a high-traffic repository won't work. You'll never be able to merge because you will be too far behind HEAD by the time they are finished (and you might bankrupt your company in the process).

#### <p style="margin: 30pt">The fact that merge quality and release quality are slightly different is actually an important realization when it comes to implementing code reviews.</p>

We can start thinking about *how much* code review is required before merging and how much is required before releasing? It might not be exactly the same amount. We can start thinking about *when* to review, not just how and what. Aiming for eventual quality consistency is often an acceptable tradeoff for increased developer velocity.

Let's get practical. To achieve rosy code reviews, you need something more than a review checklist; you need to consider your whole development and release process.

## :rose: Rosy Reviews

This is the review process that I introduced with my current employer, and we have been successfully practicing it for several years. It's used by a medium-sized department of the company with ~60 developers working in a monorepo while producing ~30 PRs/day.

It focuses on the developer experience by reducing synchronization overhead (waiting) and by minimizing flow state interruptions. As a compromise, it takes a small step away from quality consistency on the main branch, towards *eventual* quality consistency.

1. Practice shared code ownership, but don't be naive about it. Think: stewards, not owners.
2. Require *one* review, but not necessarily by the code steward (owner).
3. Encourage stewards to regularly review changes committed to components they care for.
4. Review what matters. Compile with "warnings as errors". Enforce linting and strict auto formatting of code.
5. Lean heavily on unit tests. Block PR merges until they pass. Scale CI appropriately so they run in less than 15 minutes.
6. Run additional tests on every commit after merging. Scale CI appropriately so they run in less than 30 minutes.
7. If the tests that run on trunk after the merge fail, always revert. Don't try to fix the issue while leaving the main branch broken.
8. Run the gnarly tests (expensive and/or long-running) on the main branch as often as is permissible.

## 🧙 Code Stewards, Not Owners

Code ownership is a bad idea. The owners become a synchronization barrier that slows down all other developers. It's also disruptive to the owners themselves since they need to act with low latency on review requests.

This overhead will further disincentivize developers from doing safe and minor refactorings in code that they don't "own." This is a missed opportunity for some everyday quality upkeep.

You can still require a review from *someone* to reap those sweet benefits of code reviews. Leverage locality and request the review from someone in your team, but don't ask the same person every time.

Naturally, some developers will know certain parts of the code better than others. They might even have a special responsibility for maintaining it in a good state. I call them Stewards, not Owners.

Getting a review from a steward is desirable but not required. Instead, a steward can consider changes to their component at their own pace, without low latency interruptions. They can review changes even after they have been merged to the trunk. If the steward wants a change, they request a follow-up PR to address that. Worst case, the steward can request a revert, but in our experience, that almost never happens.

If you are making significant or structural changes to a component, or if you are new to this particular area of the codebase, it's considered good etiquette to discuss the change with the steward beforehand. Leverage the CODEOWNERS feature of GitHub to get your Stewards automatically invited as lurkers to all PRs touching code that they care about.

If you find this lack of control scary, stop hiring people that you don't trust to commit changes to your code in a responsible way.

## :sparkles: Raise the Bar for Merges

This incremental reviews process must be used in combination with extensive automated testing. We are shifting some weight from reviews to testing when it comes to ensuring trunk quality.

This is not a bad thing. Reviews can focus on higher-level and big-picture issues while most of the nitty-gritty bugs get squished by the tests before anyone other than the change author is involved.

Doing "lazy" reviews like this, with optimistic merges is one of those: "You must be this tall to ride" type of things. If you don't have the tests, you don't get to do your reviews like this.

Enforcing unconditional and automated formatting before merging is also key. If you are discussing code formatting in reviews, you need to stop. Pick a formatter: ruff, clang-format, rustfmt, or whatever and get on with more important things. If you are having a hard time implementing this because of differing opinions, you have a failure of technical leadership. Fix that problem first.

## ⛽ Your Milage May Vary

The Rosy Reviews process works well for us but it might not be for everyone. Our reviews latency is lower than in many other organization, but it still too high if you ask our developers. If you are curious about going down the eventual quality consistency route, here are some of the challenges we encountered. Consider if this is something that might need special attention in your organization.

<center>![Clueless Revierer](clueless-reviewer.jpeg){ width=50% }</center>

It doesn't work without reliable tests. If you do "lazy" reviews without tests, you're back to yolo land. That's only fun until it breaks.

The reliable part of "reliable tests" is important. If you have flaky tests as a gate to pass before merging, it will drive your developers crazy with frustration. If you have flaky tests running after a merge, you can't say for sure if you should revert a commit that fails. This leaves the trunk in limbo-land between a red and green state. Developers can't trust that they can pull main with "no regrets".

Working on our tests has been, and still is, one of our major investments that makes this way of doing reviews even possible.

<pre><p style="text-align: center; margin-top: 0px; margin-bottom: 4pt;">•  •  •</p></pre>
