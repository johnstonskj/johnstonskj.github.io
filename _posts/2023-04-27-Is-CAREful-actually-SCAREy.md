---
title: "Is CAREful actually SCAREy?"
layout: postx
category: work
tags: [design]
---

I had some useful, and awkward, feedback from a colleague on my recent 
[CAREful Design Documents]({% post_url 2023-03-14-CAREful-design-documents %})
post. The feedback was basically "*I read the CARE attributes, but you always tell us the most* **important** *thing
to bring to a review is* **scope**". This is absolutely true, I do. So how does *scope* relate to CARE (henceforth the
CARE framework)?

Defining *scope* is actually answering the question *"why?"*; *why* are we doing this thing, *why* is this thing hard,
*why* do we need advice, and so forth. When presenting a design document you need to define what the system you are
describing is expected to **do**, and **not do**. In a design review you need to define what you **need** from the
reviewers, and what you **don't need**. Let's look at these questions in more detail.

It is often the case that when acting as a design reviewer we do not have context on the system we review. As such a
brief description of the scope of the system helps to determine if the system is fit for purpose. For example, a high
performance, highly scalable, six 9s system is a wonderful thing, but of the scope is to run as a batch every 24 hours
for a dozen items it's probably over engineered. I once sat in on a review for a system nicknamed *toaster*, and the
document to read just jumped right into details of the component internals. So I asked *"does your service actually make
toast, and if so how do I select the done-ness?"* This was met with blank stares, but as I have no scope, no discussion
of *why* this service needs to exist I am going to make wild assumptions. For larger design reviews, or especially
reviews for customer-facing systems I recommend the team bringing their product folks as they are often well placed to
answer this particular *why*. In this way it makes clear that documenting the scope of the system is related to the
*assumptions* attribute in the CARE framework.

Sometimes it is not possible to describe the system in a discrete manner, you have to describe some of the context
within which it operates. In this case it is really important to be clear about the scope of the system lest you find
yourself unnecessarily reviewing a related service. This form of scope can often be as simple as a block diagram with a
red border around the things that are *in scope* of the review. This use of scope, and in particular the need to
document how the in scope system relates to others, is related to the *constraints* attribute in the CARE framework.

One question that can often come up when you are reviewing large or complex systems is *why now*, or it's friend *why*
**all of it** *now*? This is a reasonable discussion which addresses urgency, dependency management, and opportunities
for iterative delivery. This is another example where product managers and development managers can be useful to
understand the natural tension between product managers tendency to want everything tomorrow, and the engineering team's
desire to make sure they maintain standards and can deliver a quality product. This use of scope is related to both the
*constraints* and *risks* attributes in the CARE framework.

Finally, at least for now, there's a useful aspect mentioned above, meeting management. As a senior/principal
engineer teams outside of your direct organization will ask for your participation in a review. This means you may not
have a lot of context to be productive in a 1-2 hour meeting. This often means we jump to our favorite cognitive tool to
understand what we hear (the data guys try to understand the data model, the systems guys are looking at runtimes, etc.)
and favorite issues to pick at. This is reasonable up to a point, it is easy to rat hole on irrelevant details as those
details jump out at us as our internal pattern matcher overfits. Now, get more than one reviewer doing the same and we
move from *rat hole* to *[rat king](https://en.wikipedia.org/wiki/Rat_king)*! I tell teams asking how to prepare for a
design review that if you have $n$ Principal Engineers (PE) in a room you can expect a **minimum** of $n+1$ opinons to
deal with as we are extremely good at arguing with ourselves. To make matters worse it is usually the case that the team
owning the review are of a lower level than the reviewers, so the dynamic makes it hard for them to keep the meeting
on-track and therefore useful. My advice is for that team to write down the scope of the meeting itself, what do you
**need** from the reviewers and what area of the system needs review. This allows even a junior member of the owning
team to redirect conversations "hey, that's not in the scope we agreed" and bring things back on track. This does also
mean that it is a responsibility of the reviewer to recognize and acquiesce to the redirect. Or, if they feel strongly
that something needs more attention, to table the discussion or propose a follow-on meeting. This use of scope relates
to the *effort* attribute in the CARE framework, respecting the effort and time of the owning team.

So I guess I need to add Scope to CARE which results in SCARE, or CARES but it doesn't feel right to have the scope at
the end, and anyway the **SCARE Framework** sounds awesome.

- *Scope* --- a *what* are we doing and *why* are we doing it. Scope should
  address:

  - **Why** are we building, or enhancing, the system. 
  - **What** the system must do and just as importantly, not do.
  - **How** will measure, or otherwise determine, the success of the system?
  - For a design review: what do you **need**, and **not need**, from reviewers.

- *Constraints* --- a set of restrictions, or limitations, within which a system must operate.
- *Assumptions* --- the set of things we think we know (but will hurt us if we're wrong).
- *Risks* --- the set of known concerns that we need to address or mitigate.
- *Effort* --- the enumeration of resources required to accomplish the design, **and** operate/maintain the resulting
  system(s).
