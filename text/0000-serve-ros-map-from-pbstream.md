# Serve ROS map from a pbstream file

## Summary
[summary]: #summary

We would like to have a ROS node that serves a `nav_msgs/OccupancyGrid` message like the `occupancy_grid_node` does.
But instead of publishing the current state of cartographer, a frozen state from a pbstream file should be read and compiled into a flat map, which is then published.


## Motivation
[motivation]: #motivation

Since `OccupancyGrid` is the standard message type for map, many nodes depend on such a topic.
If the pbstream gets frequently updated re-generating a `.pgm` and `.yaml` file in oder to read it in again using a ROS `map_server` is administrative overhead and generates redundant data. 

## Approach
[approach]: #approach

This feature will live in a new binary along with `occupancy_grid_node` and `pbstream_to_ros_map`. Existing code for deserializing protos from a pbstream file will be reused by factoring it out of `pbstream_to_ros_map` and bringing it to libcartographer.

## Discussion Points
[discussion]: #discussion
