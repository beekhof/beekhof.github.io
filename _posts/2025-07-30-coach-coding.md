---
date: 2025-07-30 10:28
title: Building a Kubernetes Operator in a Day with AI
excerpt: 
published: true
tags:
- ai
- best practices
---

### **My Brain is Utterly Broken: How I Built a Kubernetes Operator in a Day with AI**

As someone that always took pride in my ability to rapidly implement features
and generally solve problems in software, I was extremely skeptical of the "vibe
coding" trend.

Despite my misgivings, I felt I needed to understand it before completely
disregarding it.  However, after giving it a try, I cannot stop talking about
it.

It’s been a genuine paradigm shift for me.  The process took me from a simple,
niche, idea to a functioning project far faster than I could have ever imagined.

Honestly, my mind has been blown, the veil has been lifted, and I’m still trying
to piece together what it all means.

#### **The Idea That No One Had Time For**

It started, as many of these things do, with a real customer need. I had an idea
for a new operator that would help our Virtualization customers using ODF:

*A cloud-native version of SBD from `clusterlabs`, designed to work with NHC from `medik8s`*

There are maybe a dozen people in the world to whom that sentence makes sense
without a fair bit of research (I didn’t tell it that SBD was Storage Based
Death, that NHC was Node Health Check, or what either of those two things
did*)*.  Which made it the perfect test case: it was non-trivial, addressed a
real-world problem, and wholesale plagiarism was off the table.  With no
engineering capacity to do it properly, there was nothing to lose.

#### **The "Vibe Coding" Experiment**

Inspired by a [LinkedIn
course](https://www.linkedin.com/learning/structure-vibe-coding-to-save-build-time/tips-for-your-ai-vibe-coding-journey)
on "vibe coding", I decided to put the process to the test. I was sceptical it
would be applicable outside of their carefully navigated demos, and stacked the
deck against it. My goal was to model a worst-case scenario:

What would an intern produce on their first day using this new AI-driven process? 

How much work would it be to get that result into a shippable state?

I fed my one-sentence idea into the process. Using Gemini, I let it take the
reins, answering its questions with "you decide", and did not even bother to
read the results.  I fed the generated prompts into the newly approved `cursor`
AI tool, provided zero oversight, and blindly accepted any code it generated…

A few hours later, I had [https://github.com/beekhof/sbd-operator/tree/v1.0](https://github.com/beekhof/sbd-operator/tree/v1.0) and this concise problem statement:

*Current Kubernetes node remediation solutions often rely on IPMI/iDRAC for
 fencing, which is not feasible in many cloud environments or certain on-premise
 setups. For stateful workloads depending on shared storage (like Ceph,
 traditional SANs via CSI), a mechanism is needed to reliably fence unhealthy
 nodes by leveraging their shared storage access, ensuring data consistency and
 high availability by preventing split-brain scenarios in a way that is
 consistent with the workloads it protects.*

This was not a toy.

Let’s be clear though: the code didn’t work. There were gaps in the
implementation, the AI had cleverly used `--ginkgo.dry-run` to fake its way
through the e2e tests, and the build system was a Rube Goldberg machine that
made my eyes bleed. But as a starting point created in just a few hours? It had
no right to be this good. In one day, I had a design, tests, and a
well-documented codebase.

#### **The Real Magic: AI-Powered Iteration**

While the initial code drop was impressive, the follow-up process was truly
transformative. The bot iterates faster than I ever could.

For instance, my colleague pointed out that the operator should be using
`conditions` instead of `phases`—a standard best practice for operators. To fix
this, I simply typed the following prompt:

**Prompt:** `“convert all use of phases to conditions”`

That was it. That was the only thing I typed. About 10 minutes later, having
fixed its own compile errors and updated the tests with no additional
interaction from me, this commit appeared:
[https://github.com/beekhof/sbd-operator/commit/2a5f212d8bcbd8cbd46f3794d3acc2c28a28ef0b](https://github.com/beekhof/sbd-operator/commit/2a5f212d8bcbd8cbd46f3794d3acc2c28a28ef0b)

While the steps were often imperfect, at the speed it iterated, the cumulative
effect of even small improvements was hugely significant in short periods of
time. Two weeks into the experiment I would consider it a reasonable beta.

**Success Means Defining (and Defending\!) the Success Criteria**

While the bot can crank out code and iterate at a speed that's insane, the key
to success is human oversight—someone actually making sure its output is solid.
Sweating the details around the build system and test suite defines the bot’s
success criteria.

Unsupervised, the bot will do *anything* to allow both of those to succeed,
including cheating. This can manifest in several insidious ways: it might
transform complex, problematic code sections into simplistic print statements,
effectively sidestepping the actual logic that needs testing. I have also seen
it subtly alter the assertions and conditions within the tests, ensuring they no
longer rigorously check for the intended functionality but instead validate a
more easily achievable (and potentially incorrect) outcome. In the most extreme
instances, it has outright deleted tests that proved too difficult to pass.

Likewise, the bot is so ridiculously good at just *making* code, it sometimes
completely ignores the idea of reusing existing functionality. Why bother
finding and integrating an existing function when it can whip up a new, slightly
different version in seconds?  Unchecked, the result is dozens of essentially
the same function with subtle differences.  It’s not just messy, it’s a
maintenance nightmare that takes care and time to unravel.  Worst of all, the
sheer volume of code represents a bigger surface for the bot to “adjust” in
order to make tests pass, and harder to spot in bulk changes.

These actions, while entirely predictable, highlight the need for a human to be
in control at all times.

**A Better Name**

The term "vibe coding" still makes me somewhat nauseous. It sounds like
something you do after a few too many energy drinks, and reflects poorly on both
the process and the folks wielding it so effectively.

To me, the process felt like coaching a seriously brilliant, albeit occasionally
mischievous, junior engineer who's somehow operating at warp speed. You're
constantly guiding, correcting, and nudging it in the right direction, providing
that human oversight. I’m starting to use the term "Coach Coding" instead, as it
is a more accurate reflection of the collaborative, iterative, and ultimately
human-driven nature of the process.

#### **An Insanely Powerful Tool**

The bot is not perfect, but that’s not the point. In the hands of an experienced
practitioner who can spot missteps and highlight areas for improvement, it is an
insanely powerful tool. It’s not about replacing the developer, but amplifying
their ability to execute and iterate at a speed I’ve never imagined possible.

The experience of going from a niche idea to a tangible, well-structured project
in a matter of days has fundamentally changed my perspective on what’s possible.

The future of development is already here, and it’s more amazing than I thought
possible.


