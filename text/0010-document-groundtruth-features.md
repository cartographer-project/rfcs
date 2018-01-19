# Documentation for Ground Truth Features

## Summary
[summary]: #summary

Document the `cartographer_autogenerate_ground_truth` and `cartographer_compute_relations_metrics` executables better.

## Motivation
[motivation]: #motivation

The "ground truth" evaluation binaries are currently a bit of a hidden feature that is primarily used for the CI pipeline.
But they are probably also useful for other people, e.g. for quality assessment while tuning.
Currently, users need to consult the source code to find and understand them.
A documentation could raise awareness for this feature and can make it more convenient to use them.

## Approach
[approach]: #approach

Add a new file with a brief concept explanation, use cases and a dummy example (like the one shown in [open house June 22, 2017](https://storage.googleapis.com/cartographer-public-data/cartographer-open-house/170622/sildes.pdf)) to `docs/`.

## Discussion Points
[discussion]: #discussion

tbd
