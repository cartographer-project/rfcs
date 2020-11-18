# Delete / load interface in Cartographer ROS

## Summary
[summary]: #summary

Expose the delete / load interface of the pose graph in Cartographer ROS.

## Motivation
[motivation]: #motivation

The pose graph / map builder interface supports deleting trajectories and loading states at runtime.
The motivation of RFC-23 for why delete/load is useful also applies here.

So far, we have gRPC services for doing that:
* [DeleteTrajectory](https://github.com/googlecartographer/cartographer/blob/master/cartographer/cloud/proto/map_builder_service.proto#L111)
* [LoadStateFromFile](https://github.com/googlecartographer/cartographer/blob/897762675caffdb8f21d07ff01824528b56c2e8b/cartographer/cloud/proto/map_builder_service.proto#L161)


The goal of this RFC is to make this accesible from Cartographer ROS.


## Approach
[approach]: #approach

### New ROS services

`/delete_trajectory`:
```
int32 trajectory_id
---
cartographer_ros_msgs/StatusResponse status
```

`/load_state_from_file`:
```
string file_path
bool load_frozen_state
---
cartographer_ros_msgs/StatusResponse status
```

### Handlers

`HandleDeleteTrajectory` needs to:
* check the requested trajectory ID exists and is in trajectory state `FINISHED` or `FROZEN` (otherwise [the deletion work item crashes](https://github.com/googlecartographer/cartographer/blob/master/cartographer/mapping/internal/2d/pose_graph_2d.cc#L613)) - [CheckTrajectoryState](https://github.com/googlecartographer/cartographer_ros/pull/1262) will be useful for that
* call `DeleteTrajectory`

`HandleLoadStateFromFile` just calls `LoadStateFromFile`.

## Discussion Points
[discussion]: #discussion

### Potential race conditions, necessary checks

* deletion is handled by a trimmer, which means it is delayed until the next optimization
* deletion of frozen trajectories requires a patch, see: https://github.com/cartographer-project/cartographer/pull/1767
