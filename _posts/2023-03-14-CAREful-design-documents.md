---
title: "CAREful Design Documents"
layout: postx
category: work
tags: [design]
---

Let's face it, as a senior engineer a common task is to lead design
reviews at different stages of a system's life-cycle. This inevitably
leads to questions about documenting designs, and specifically design
decisions. Over time I've come to realize that at each decision point,
or discussion of a feature, I find myself asking for the same set of
information.

However, recently as I write another *design review guidelines* document
I realized that this common set of four topics comes with a rather
helpful acronym – CARE. So why should you, the reader, *care* about my
neat realization? If nothing else, I would bet you already consider
these items in technical discussions and so you get to give it a name.

# What is CARE?

- *Constraints* – a set of restrictions, or limitations, within which a
  system must operate. For example:

  - The system **must** operate within pre-defined SLA's (latency,
    availability, …).
  - The system **must not** increase operational costs.
  - The system **must** be operable on a set of platforms.
  - The system **must** pass security review(s).
  - The system **must** launch within a given time-frame.

- *Assumptions* – the set of things we think we know, which typically
  fall into the following categories.

  - The way **we think** something works.
  - The way someone **told us** that something works; which in turn may
    be base only on what that someone thinks it works, and so on.
  - What a user **said that** they want.
  - A set of beliefs we take for granted as facts (Quicksort always
    wins).
  - A set of biases based upon much of the above, but also including
    inter-personal and unconscious biases.

- *Risks* – the set of known concerns that . It is common for risks to derive
  from the last two items; for example, some constraints may require a more
  complex design than would otherwise suffice, or an assumption may prove
  untrue and cause re-work of a design. Risks may be non-technical, and we do
  have a professional obligation (see [Professional
  Practice](#professional-practice)) to ensure that we raise those as we see
  them.

- *Effort* – the enumeration of resources required to accomplish the design,
  **and** operate/maintain the resulting system(s). While it may make sense to
  expand *effort* to include *cost* I believe this is invalid. If a cost
  budget, or calendar deadline is the focus of the design it is a
  *constraint*. This does not mean we do not consider costs, it should always
  be a part of our [professional practice](#professional-practice) to
  carefully balance the costs (and other impacts) of our work.

# Using CARE in design decisions

When faced with a major design decision it is common to document them in
a simple manner:

- Problem – what's new or broken.
- Potential Solutions – a set of choices, with pros and cons.
- Decision – what did you decide, and why (when you get to that point).

CARE can apply in each of these sections, although the focus is somewhat
different in each.

- Problem – What constraints does the existing system impose on our
  choices? What assumptions do we have about the current state? What
  risks exist in the current system? What effort are we expending to
  keep the current status quo?d
- Potential Solutions – What additional constraints do they impose on
  us? What assumptions do we have about the solution and how it works?
  What risks do we see with the solution, and what effort will we need
  to expend to use it?
- Decision – The CARE attributes here are usually those associated with
  the option expressed in the *potential solutions*.

# Using CARE in discussions

While any one of the CARE attributes may actually be the focus of a
technical discussion, I tend to use them as a framework to ensure I've
looked at things from all four perspectives.

If nothing else, I believe they provide a good set of terms and
terminology, both the four attributes as well as a set of derived terms.
We have already discussed how constraints include SLAs, costs, time, and
so forth. In a similar way assumptions may be fundamental beliefs,
biases, or transient beliefs – this system is way faster than that one.

In the case of risks and effort, the terminology used will be much the
same but applied in a different manner.

# Professional Practice

From the ACM [Software Engineering Code of Ethics and Professional
Practice](https://ethics.acm.org/code-of-ethics/software-engineering-code/),
Principal 3 (PRODUCT):

> 3.01. Strive for high quality, acceptable cost and a reasonable
> schedule, ensuring significant tradeoffs are clear to and accepted by
> the employer and the client, and are available for consideration by
> the user and the public.

> 3.09. Ensure realistic quantitative estimates of cost, scheduling,
> personnel, quality and outcomes on any project on which they work or
> propose to work and provide an uncertainty assessment of these
> estimates.

Principal 8 (SELF):

> 8.02. Improve their ability to create safe, reliable, and useful
> quality software at reasonable cost and within a reasonable time.
