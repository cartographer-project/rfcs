# Simplify start_trajectory.srv

## Summary
[summary]: #summary

This RFC summarizes a few observed problems _of_ and suggestions for improvement _for_ the `/start_trajectory` service of Cartographer ROS.

## Motivation
[motivation]: #motivation

Starting a trajectory on demand, optionally with a start pose, is a very common task in robot localization.
However, the [`/start_trajectory`](https://github.com/googlecartographer/cartographer_ros/blob/3ca30fc90458152cc9a2c52aaa556e0ab09d0871/cartographer_ros_msgs/srv/StartTrajectory.srv) service is very hard to use for service clients. 

Recent discussions related to this:

* https://github.com/googlecartographer/cartographer_ros/issues/1193
* https://github.com/googlecartographer/cartographer_ros/issues/1212
* https://github.com/googlecartographer/rfcs/pull/17#issuecomment-468639993

It requires a binary string of the trajectory options proto (see [here](https://github.com/googlecartographer/cartographer_ros/blob/61dd57bd94/cartographer_ros_msgs/msg/TrajectoryOptions.msg)), which is impossible to build by client code that should only know the message contract from `cartographer_ros_msgs`.

The `start_trajectory_main.cc` executable exists only to make this service usable.
It hides the implementation details away from the user/client with a proper interface that has only 3 simple options.

## Approach
[approach]: #approach

Change the service definition so that clients only need to depend on message packages (`cartographer_ros_msgs`, `geometry_msgs`, `std_msgs`), e.g.:

```
string configuration_directory
string configuration_basename
bool use_initial_pose
geometry_msgs/Pose initial_pose
int32 to_trajectory_id
---
cartographer_ros_msgs/StatusResponse status
int32 trajectory_id
```

Then, change the service handler in `node.cc` to do what `start_trajectory_main.cc` has been doing so far (essentially: loading the configuration).

As a nice side effect we can get rid of the following:

* `start_trajectory_main.cc`
* `TrajectoryOptions.msg`
* ROS message conversions in `trajectory_options.h`
* `SensorTopics.msg`

## Discussion Points
[discussion]: #discussion

### Optional initial pose

It should be possible to start a trajectory without initial pose, just as now when the initial pose argument isn't given to the start_trajectory executable.

Unlike e.g. protobuf, there's no way to define a part of a message definition as optional in ROS.
`to_trajectory_id` would be zero initialized if unspecified, which is a valid trajectory ID.
Maybe it's possible to find a workaround:

* hide the pose in an array that can have zero elements (ugly)
* two services: `/start_trajectory_with_initial_pose`, `start_trajectory`
* ...?

**Decision**:
Add boolean message field `use_initial_pose` - easy to understand and easy to use.

### Keep time parameter?

The time parameter of the initial pose is a bit of an obscure feature that might confuse people - should we keep it in the ROS service? See [cartographer/pose_graph.h](https://github.com/googlecartographer/cartographer/blob/bdb6f2db4a1f98484b222d61abceab8adb74dfd1/cartographer/mapping/pose_graph.h#L138)

**Decision**:
Don't expose in ROS service.

### Advantages of passing options in the message?

As mentioned by @ojura in the open house meeting, the proposed approach requires that the configuration lies on the host filesystem of the Cartographer node.
Binary proto allowed to transfer options from a client in the network.

**Decision**:
We agreed that we will drop this possibility for the sake of simplicity, which will benefit more users.
Also, client/server scenarios are better covered by Cartographer gRPC anyway.

