# Introduce [Timed]RangefinderPoint

## Summary
[summary]: #summary

Introduce `[Timed]RangefinderPoint` to be used throughout Cartographer instead of using `Eigen::Vector3/4` directly.

## Motivation
[motivation]: #motivation

Currently, Cartographer is using `Eigen::Vector3/4` directly in lots of places.
For example, a rangefinder point cloud is an `std::vector` of `Eigen::Vector3/4`.
Suppose one wants to extend rangefinder points passing through Cartographer to include some metadata (e.g. laser ID).
Currently, that's not possible, but if Cartographer consistently used a structure named e.g. `RangefinderPoint` for rangefinder points instead of `Eigen::Vector3`, it would be trivial for the user to extend Cartographer to do this. 

## Approach
[approach]: #approach

Introduce classes `[Timed]RangefinderPoint` and replace all uses of `Vector3/4` with the new classes.

Slicing `TimedRangefinderPoint` to `RangefinderPoint` shall be used instead of calling `Vector4::head<3>`. 

Define `operator*(Rigid3, RangefinderPoint)` for e.g. tracking <-> local frame transformations.

Take care not to break the data flow by avoiding constructing entirely new points - always use metadata-preserving functions on incoming points.

Libcartographer users could then easily modify `RangefinderPoint` to include their metadata.

## Discussion Points
[discussion]: #discussion

Inheritance of `Vector3/4` instead of composition?
Answer: composition/

Class naming?

Proto compatibility?
Answer: Retain. 
Will rename existing `Vector3/4` proto fields to `*_legacy` and make sure old .pbstreams can be deserialized by adding appropriate logic to `FromProto([TimedPointCloud|Range]Data)`.

Implementation available in [googlecartographer/cartographer#1357](https://github.com/googlecartographer/cartographer/pull/1357).
