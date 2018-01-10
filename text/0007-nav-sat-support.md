# NavSatFix (GPS) Support in Cartographer ROS

## Summary
[summary]: #summary

The goal is to add [NavSatFix](http://docs.ros.org/api/sensor_msgs/html/msg/NavSatFix.html)  (GPS) support to Cartographer ROS. 
The NavSatFix (GPS) data will be used in the pose graph optimization, using the fixed frame pose feature of Cartographer.

## Motivation
[motivation]: #motivation

In outdoor applications, especially cars, GPS data is commonly available. 
Using this data will reduce the drift in location, especially over large distances. 

## Approach
[approach]: #approach

We will wire up the [sensor_msgs/NavSatFix](http://docs.ros.org/api/sensor_msgs/html/msg/NavSatFix.html) and introduce two new options for this: `use_nav_sat` and `fixed_frame_pose_sampling_ratio`.

These messages will be [translated](https://en.wikipedia.org/wiki/Geographic_coordinate_conversion#From_geodetic_to_ECEF_coordinates) first into a common cartesian coordinate frame, [ECEF](https://en.wikipedia.org/wiki/ECEF). 

Then we will transform to a local ground reference frame (e.g. [ENU](https://en.wikipedia.org/wiki/Axes_conventions#Ground_reference_frames:_ENU_and_NED)) with the z-axis pointing up, and the origin at the start position.

The result will be given to Cartographer as `FixedFramePoseData`.

## Discussion Points
[discussion]: #discussion

### Should we implement a generic fixed frame pose message instead?

This RFC is specific to adding GPS-like data from statellite based positioning systems. 
We use the `NavSatFix` message which is widely used in the ROS community, and convert it from longitude, latitude, altitude to a cartesian coordinate system.
In addition, in a separate RFC, a generic fixed frame pose message could also be supported.

### Why don't we use ECEF? 
In Cartographer, the z axis points up. We want to preserve that property also in the fixed frame pose. 
