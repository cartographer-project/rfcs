# Unify TrajectoryBuilder Interfaces.

## Summary
[summary]: #summary

This RFC proposes the introduction of a unified interface for trajectory builders in the public facing API.

## Motivation
[motivation]: #motivation

Currently we have `TrajectoryBuilder` and `GlobalTrajectoryBuilderInterface` which are functionally equivalent except for the `sensor_id` which is passed to `TrajectoryBuilder` methods but not `GlobalTrajectoryBuilderInterface`.
The only implementation of `TrajectoryBuilder` is the `CollatedTrajectoryBuilder` and the only implementation of `GlobalTrajectoryBuilderInterface` is `GlobalTrajectoryBuilder`.
Since these interfaces are so similar that they are hard to tell apart we can simplify the public API by unifying them.

`sensor::Data` offers a dispatch mechanism which allows `CollatedTrajectoryBuilder` to store them in a unified way and dispatch them to the `GlobalTrajectoryBuilderInterface`.
For [cloud-based mapping](text/0002-cloud-based-mapping-1.md) we have incoming `sensor::Data` from the N gRPC threads.
This sensor data needs to be stored in a queue waiting for the SLAM thread to process them.
Unifying the interfaces allows to reuse the dispatch mechanism to feed the sensor data to the `CollatedTrajectoryBuilder`.

## Approach
[approach]: #approach

A proposed implementation is available in [#736](https://github.com/googlecartographer/cartographer/pull/736). TL;DR:

1. Let `GlobalTrajectoryBuilder` implement `TrajectoryBuilderInterface`.
2. Let `CollatedTrajectoryBuilder` implement `TrajectoryBuilderInterface`.
3. `TrajectoryBuilderInterface` is largely equivalent to `GlobalTrajectoryBuilderInterface` except that `sensor_id` is now passed to the various `AddSensorData` methods.

## Discussion Points
[discussion]: #discussion

