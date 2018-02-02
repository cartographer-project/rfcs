# Improve ROS service responses

## Summary
[summary]: #summary

Improve `cartographer_ros`' services to have more descriptive responses to implement the request/response philosophy of ROS services better.

## Motivation
[motivation]: #motivation

The motivation for ROS services is to allow *RPC request / reply interactions* (see [ROS docs](http://wiki.ros.org/Services)).
`cartographer_ros` offers such services, however the currently implemented behaviour is not always transparent for the service caller.

**An example of the current behaviour:**
1. start a trajectory (ID: 0)
2. intentionally call the wrong service: `rosservice call /finish_trajectory "trajectory_id: 666"`

The node's log will display the reason for the expected error:
```
[... node.cc:405] Trajectory_id 666 is not created yet.
```
However, the service caller gets a non-descriptive, empty message that the service call failed:
```bash
ERROR: service [/finish_trajectory] responded with an error: 
# [sic!]
```

The reason for this behaviour is that the `HandleFinishTrajectory` callback returns false if there is a logical error:
```cpp
bool Node::HandleFinishTrajectory(
    ::cartographer_ros_msgs::FinishTrajectory::Request& request,
    ::cartographer_ros_msgs::FinishTrajectory::Response& response) {
  carto::common::MutexLocker lock(&mutex_);
  return FinishTrajectoryUnderLock(request.trajectory_id);
}
```

A `false` return value, resulting in a failed service call, does not indicate for the caller whether it was a logical error (here e.g. wrong trajectory ID) or if something in the communication failed (see also [this discussion in the ROS forums](https://answers.ros.org/question/268051/why-isnt-the-response-filled-if-my-service-returns-false/?answer=268100#post-id-268100)).
As a result, the failed service calls induced by `false` return values have no useful information for a caller that can't access the logs directly.


## Approach
[approach]: #approach

A cleaner way would be to let the callbacks return always `true` to indicate a service was properly *executed*, but at the same to have additional response fields that indicate for the caller whether the service *did the right thing*.

The affected callbacks are:
* `HandleSubmapQuery` ([src](https://github.com/googlecartographer/cartographer_ros/blob/2538ac3e45ccaf553e956bbb7e745c26008460bf/cartographer_ros/cartographer_ros/node.cc#L129))
* `HandleStartTrajectory` ([src](https://github.com/googlecartographer/cartographer_ros/blob/2538ac3e45ccaf553e956bbb7e745c26008460bf/cartographer_ros/cartographer_ros/node.cc#L427))
* `HandleFinishTrajectory` ([src](https://github.com/googlecartographer/cartographer_ros/blob/2538ac3e45ccaf553e956bbb7e745c26008460bf/cartographer_ros/cartographer_ros/node.cc#L485))
* maybe also `HandleWriteState` ([src](https://github.com/googlecartographer/cartographer_ros/blob/2538ac3e45ccaf553e956bbb7e745c26008460bf/cartographer_ros/cartographer_ros/node.cc#L492))

## Discussion Points
[discussion]: #discussion

### Response data types

Ideally, the caller gets a response that makes the outcome of the callback transparent.

**gRPC Error Code and Message String** <br>
Follow [error ID scheme of gRPC](https://developers.google.com/maps-booking/reference/grpc-api/status_codes) and add a detailed status message.

Template `.srv` implementing this approach:
```
# request params
---
uint8 status_code  # 0..15 as in gRPC definition
string status_msg
# other response params
```

The `status_msg` should ideally be equal to the log messages.
The two status response field could also be wrapped inside a single, custom `ServiceStatus` message.


~~**Success Flag and Message String**~~
<details>
<summary>Click to expand discarded proposal</summary>

* pro:
  * easy to implement
  * boolean flag clearly indicates success / failure
  * message easy to interpret for user
* contra:
  * calling code might not be able to interpret the message
</details>

~~**Error Code + Dictionary**~~ <br>
<details>
<summary>Click to expand discarded proposal</summary>

A numeric return value which can be used to look up the detailed error description in a common error dictionary.
Alternatively, [enum-like behaviour](https://answers.ros.org/question/9427/enum-in-msg/?answer=105806#post-id-105806) could be implemented in the message definition.
* pro:
  * easy to implement
  * service caller can look up errors from the dictionary without knowing the intrinsic behaviour of the callback, e.g. for logging
* contra:
  * error dictionaries of the caller and the handling node must not diverge
</details>


### Others

* any other affected services that are not listed in [Approach](#approach)?
* any other ideas for response data types?


