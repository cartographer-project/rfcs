# Replace frozen trajectory from file

## Summary
[summary]: #summary

We propose to add `PoseGraphInterface::DeleteTrajectory`, `PoseGraphInterface::GetTrajectoryStates`, and `MapBuilderInterface::LoadStateFromFile` because these functions are missing to replace a frozen trajectory ("map") at run-time.

## Motivation
[motivation]: #motivation

Currently, it is not possible to replace a finished (and frozen) trajectory at run-time.
In fact, the trimmers are the only components that even delete parts of the pose graph.
When we add these two functions to the public API, the use case of swapping a frozen map at run-time is possible.
A caller can load a new frozen map, wait for global SLAM to process it (and re-localize in the new map), and delete the old frozen map.
Another use case is to clean up old trajectories on a device that runs continuously and sometimes starts and finishes trajectories.

## Approach
[approach]: #approach

In `PoseGraphInterface`, we add `virtual void DeleteTrajectory(int trajectory_id) = 0;`.
The deletion is delayed until previously scheduled work items have finished and a constraint search phase is completed.
This is necessary to avoid use-after-delete conflicts and races.
While a trajectory is scheduled for deletion, `Add*Data` operations are skipped and reported as warnings.
Only finished or frozen trajectories can be deleted.

We also manage the states of a trajectory more clearly, introducing the states:

```cpp
enum class TrajectoryState { ACTIVE, FINISHED, FROZEN, DELETED };
```

Possible transitions are from `ACTIVE` to `FINISHED` or to `FROZEN`, and from `FINISHED` or `FROZEN` to `DELETED`.
Note that a `trajectory_id` is not re-used, so `DELETED` is a final state.

`PoseGraphInterface::GetTrajectoryStates` is added to get a map of all trajectories' states.

In `MapBuilderInterface`, we add `virtual void LoadFrozenStateFromFile(const std::string& filename) = 0;`.
Contrary to `LoadState`, the handler reads the file, so it can be called by stubs written in languages other than C++ and there is less gRPC traffic.

For the `Load*` functions, we propose to return maps which trajectories have been loaded.
This is necessary so that callers can later delete loaded trajectories.

We also add all gRPC endpoints of these new functions and a test that actually deletes a finished trajectory.


## Discussion Points
[discussion]: #discussion

