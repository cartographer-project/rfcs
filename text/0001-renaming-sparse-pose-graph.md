# Renaming the Sparse Pose Graph

## Summary
[summary]: #summary

The goal of this change is to improve readability of the code base by choosing a better name for the "sparse pose graph", e.g. "pose graph".

## Motivation
[motivation]: #motivation

Renaming "sparse pose graph" to just "pose graph" conveys almost the same information, i.e. the word "sparse" is mostly noise.
The concept of a pose graph is well known in SLAM.
Of all the features that this pose graph has being "sparse" is not particularly extraordinary.
The expected outcome is a more readable code base.
We want to do this change before version 1.0 while we do not guarantee stable option names since it will be harder to do afterwards.

## Approach
[approach]: #approach

- `sparse_pose_graph` directories are renamed `pose_graph`.
- `SparsePoseGraph` classes are renamed `PoseGraph`.
- `sparse_pose_graph.lua` is renamed `pose_graph.lua` and the word `sparse` is dropped from the options name.
- dependent configuration files (including other repositories) have to be updated to use the new options name.

## Discussion Points
[discussion]: #discussion

### Is it not an advantage that Sparse Pose Graph is a reference to Sparse Pose Adjustment?

Sparse Pose Adjustment is an algorithm for optimizing a pose graph using a direct linear solver and inspired the current implementation in Cartographer.
This is an implementation detail and might change in the future.
Having a name that sounds as if it refers to the current inspiration of the implementation thus is an argument for changing the name.

### Isn't a sparse pose a well known concept in SLAM?

No.
The name refers to the sparsity of the graph not the pose.
Removing "sparse" also gets rid of this misconception.
A pose graph is a well known concept in SLAM, e.g. in the Sparse Pose Adjustment paper, 8 of the 9 mentions of a pose graph do not qualify it as "sparse".
