# Cartographer RFCs
[Cartographer RFCs]: #cartographer-rfcs

For changes more substantial than simple bugfixes and documentation improvements
we ask that these be put through a very lightweight design review process.

"RFC" stands for "request for comments" and are used, e.g. by
[rust-lang](https://github.com/rust-lang/rfcs), to establish a consistent
and controlled path for new features to enter their project, so that new
contributors can anticipate the short- and long-term development roadmap of the
project.

## Table of Contents
[Table of Contents]: #table-of-contents

  - [Introduction](#cartographer-rfcs)
  - [Table of Contents]
  - [When should you write an RFC]
  - [Motivation for RFCs]
  - [Before creating an RFC]
  - [What the process is]

## When should you write an RFC
[When should you write an RFC]: #when-should-you-write-an-rfc

You should provide an RFC if you intend to make a *significant* change or
contribution to one of Cartographer's repositories. A change is e.g.
*significant* if it introduces a new or modifies an existing *concept*. The
process is really very lightweight so we encourage you to err on the side of
RFCs.

Make sure to have *one sentence per line* in the RFC as it makes reading diffs
much easier. There is no characters per line limit for RFCs.

## Motivation for RFCs
[Motivation for RFCs]: #motivation-for-rfcs

Before the introduction of RFCs reviewer's would sometimes spend time reading
incoming PRs trying to find out what the problem is the author is actually
solving. This could slow down reviews considerably which is unfortunate for both
the reviewer and contributor.

Sometimes a PR could sit in the review queue for a long time as it was modifying
code that was undergoing a refactoring and merging it would make that
refactoring harder. So the purpose of a RFC is both to vet incoming feature
proposals and also allow other contributors to learn about on-going projects
and coordinate better.

## Before creating an RFC
[Before creating an RFC]: #before-creating-an-rfc

Before opening an RFC we ask you to briefly look over the list of RFCs being
discussed or [implemented][Cartographer Projects]. If your RFC conflicts with
another RFC please inject yourself into the discussion by either commenting on
the RFCs PR or opening an issue about the RFC being implemented in this
repository.

## What the process is
[What the process is]: #what-the-process-is

Follow this process to get your feature merged into Cartographer.

  - Fork the RFC repo [RFC repository]
  - Copy `0000-template.md` to `text/0000-my-feature.md` (where "my-feature" is
    descriptive. Don't assign an RFC number yet).
  - Fill in the RFC.
  - Submit a pull request. As a pull request the RFC will receive design
    feedback from the larger community, and the author should be prepared to
    revise it in response.
  - When consensus about the RFC has been reached the corresponding PR will be
    approved and the PR merged.
  - The person that actually merges the PR will create a new *project* for the
    duration of the implementation (which might span several PRs) in the
    [projects pane][Cartographer Projects] of the Cartographer repository and
    assign an RFC number.
  - When you create implementing PRs, please add a link to the RFC in the PR
    description.

[Cartographer Projects]: https://github.com/googlecartographer/cartographer/projects
[RFC repository]: https://github.com/googlecartographer/rfcs
