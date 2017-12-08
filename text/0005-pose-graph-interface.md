# Introduce PoseGraphInterface to define public PoseGraph API.

## Summary
[summary]: #summary

We propose to introduce a new interface, i.e. `PoseGraphInterface`, which is a strict subset of `mapping::PoseGraph`.

## Motivation
[motivation]: #motivation

`mapping::PoseGraph` has a large interface, i.e. 24 public methods.
`mapping::PoseGraph` is exposed by `MapBuilder` through its public member function `pose_graph` which returns a pointer to a `PoseGraph`.
A brief review of clients of `MapBuilder` shows that only a small number of  methods are actually accessed throught this pointer.
These methods are

- `constraints()`
- `GetAllSubmapData()`
- `GetLocalToGlobalTransform(trajectory_id)`
- `GetTrajectoryNodes()`
- `RunFinalOptimization()`

Reducing the size of the `PoseGraph` interface exported by `MapBuilder` will make the endeavour of gRPC-ing `MapBuilder` much less cumbersome.

## Approach
[approach]: #approach

The approach is outlined in [PR#744](https://github.com/googlecartographer/cartographer/pull/744).
We introduce a new interface `PoseGraphInterface` which only exposes the above methods as pure virtual.
`PoseGraph` derives from `PoseGraphInterface`.
No changes to `cartographer_ros` are necessary.

## Discussion Points
[discussion]: #discussion

