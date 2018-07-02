# Use trajectory state in cartographer_ros

## Summary
[summary]: #summary

Make meaningful use of the new `TrajectoryState` enum in `cartographer_ros`.

## Motivation
[motivation]: #motivation

libcartographer now provides `enum class TrajectoryState { ACTIVE, FINISHED, FROZEN, DELETED };` through `PoseGraphInterface::GetTrajectoryStates`.

1. We can use this interface in `cartographer_ros` at least to simplify some existing logics.

2. Furthermore, a query for the trajectory states would be super useful for controlling the system from outside.
Up to now, collecting the active trajectory IDs via ROS in a systematic way was only possible with some workarounds.
E.g. listening to the submap list topic and extracting the IDs from there, which has unnecessary overhead.
The additional information about the state of each ID was not available yet without custom hacks, but is very valuable for systems that need to perform trajectory starts and finishes (semi-)automatically.
An easy example is handling an initial pose request in a pure localization setting: first, the current active trajectory needs to be determined and finished in order to then start a new one afterwards.

## Approach
[approach]: #approach

1. Use `GetTrajectoryStates` in existing code that used some other ways of determining the state before.

2. Introduce a `get_trajectory_states` service to query the states.
The response contains a list of IDs, a list of states and a standard header with timestamp.

## Discussion Points
[discussion]: #discussion
~~`cartographer_ros`' map builder bridge has its own `TrajectoryState` and `GetTrajectoryStates`.~~
~~These are __not__ related to the new pose graph interface of libcartographer that uses the same names.~~
~~Since they refer to local data only, a renaming makes sense even if this RFC is not considered.~~
(done, see [this pull request](https://github.com/googlecartographer/cartographer_ros/pull/908))

## To-Do

Collect code sections where the new pose graph interface could be used:
 
* add `MapBuilderBridge::GetTrajectoryStates()`
* add `Node::HandleGetTrajectoryStates(...)` as the callback for the `/get_trajectory_states` service
* `MapBuilderBridge::GetFrozenTrajectoryIds()` can be removed, was only used in one callback that check if a trajectory ID is frozen.
This can be replaced with `map_builder_bridge_.GetTrajectoryStates().at(id) == TrajectoryState::FROZEN`.
* `is_active_trajectory_` can be removed
