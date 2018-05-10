# Global SLAM Result Callback

## Summary
[summary]: #summary

For local SLAM a `LocalSlamResultCallback` can be registered when creating new trajectories which is invoked whenever new `RangefinderData` is added to the trajectory.
Similarly we propose to add a `GlobalSlamResultCallback` which is invoked whenever the optimization problem has been solved.

## Motivation
[motivation]: #motivation

[Issue #762](https://github.com/googlecartographer/cartographer_ros/issues/762) describes bandwidth issues when publishing the `/trajectory_node_list`.
To remedy this [Issue #1022](https://github.com/googlecartographer/cartographer/issues/1022) proposes the introduction of a mechanism to signal the completion of an optimization problem.
The introduction of a `GlobalSlamResultCallback` would accomplish that.

Moreover for cloud-based mapping we would like to the ability to stream the robots pose to the cloud.
With the introduction of a `GlobalSlamResultCallback` this code would look much more natural as upload would occur as a result of the callback invocation.

## Approach
[approach]: #approach

We propose to introduce a callback with the following signature:

```
using GlobalSlamResultCallback = std::function<void(
    const std::map<int /* trajectory_id */, SubmapId>&,
    const std::map<int /* trajectory_id */, NodeId>&)>;
```

i.e. the callback gets passed two maps, mapping trajectory IDs to the last optimized `SubmapId` and `NodeId`.

We would change the public API by adding the `GlobalSlamResultCallback` as a constructor argument to `MapBuilder` which passes it on as a const'or argument to either `PoseGraph2d` or `PoseGraph3d`.
The callback is invoked in `WhenDone` callback right after `PoseGraph2/3d::RunOptimization()`. We need to take a mutex lock to determine the last optimized `NodeId` and `SubmapId`, but we will give up the lock before invoking the callback to ensure that the provided callback can access the `PoseGraph`.

## Discussion Points
[discussion]: #discussion

