# Customizable Collators

## Summary
[summary]: #summary

We propose to introduce a `sensor::CollatorInterface` and add two implementations of the same:

1. one which sorts sensor data across sensor data across all trajectories (this is what the current implementation, `Collator`, does).
2. one which sorts sensor data only within the trajectory they belong to, allowing other trajectories to make progress even if one is blocked.

## Motivation
[motivation]: #motivation

For cloud-based mapping we have to deal with the situation where the robot to cloud uplink is unrealible.
A robot might venture into an area where no Wi-Fi signal is available thereby severing the cloud uplink.
The collator in the Cartographer cloud instance would currently stall as it is waiting for data from that robot before allowing any other trajectories to make progress.
Ideally this behavior would be configurable through `MapBuilderOptions`.

## Approach
[approach]: #approach

We propose to introduce a `sensor::CollatorInterface` which would be a pure virtualization of the current `Collator`.
The current `Collator` would be renamed to `AcrossTrajectoryCollator` and be marked as an implementation of that interface.
A new `Collator` would be introduce, `PerTrajectoryCollator`, which only collates sensor data belonging to the same trajectory.
This collator would have map of `OrderedMultiQueues` - one for each trajectory.
Note that using the `PerTrajectoryCollator` means that there is no longer a deterministic mapping result given the same input data.
The collators are instantiated in the `MapBuilder`.
We introduce a new option in `MapBuilderOptions` to determine which of the two implementations to instantiate.

## Discussion Points
[discussion]: #discussion

