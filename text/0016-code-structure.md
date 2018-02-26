# Restructuring Source Code

## Summary
[summary]: #summary

1. The code that lives in `"cartographer_grpc/"` and `"cartographer/"` directories will be merged.
The namespace `"cartographer_grpc::`" will be merged with `"cartographer::"` namespace.
1. Directories `"mapping_2d"` and `"mapping_3d"` will become `"/mapping/2d"` and `"/mapping/3d"`.
The corresponding namespaces will be merged with `"mapping::"` namespace.
1. Classes and data structures names will have `2D` or `3D` suffix if necessary.


## Motivation
[motivation]: #motivation

The code in `"cartographer_grpc/"` directory has passed its prototype stage.
Also this directory contains mocks for `PoseGraph`, `MapBuilder`, `TrajectoryBuilder` that can (and should) be used in the `"cartographer/"`.
Namespace `cartographer_grpc::` introduces unnecessary lengthy names.

Example from `cartographer/cartographer_grpc/testing/mock_map_builder.h `:
```
class MockMapBuilder : public cartographer::mapping::MapBuilderInterface {
 public:
  MOCK_METHOD3(
      AddTrajectoryBuilder,
      int(const std::set<SensorId> &expected_sensor_ids,
          const cartographer::mapping::proto::TrajectoryBuilderOptions
              &trajectory_options,
          cartographer::mapping::MapBuilderInterface::LocalSlamResultCallback
              local_slam_result_callback));
  MOCK_METHOD1(AddTrajectoryForDeserialization,
               int(const cartographer::mapping::proto::
                       TrajectoryBuilderOptionsWithSensorIds
                           &options_with_sensor_ids_proto));
```
instead of
```
class MockMapBuilder : public MapBuilderInterface {
  public:
   MOCK_METHOD3(
       AddTrajectoryBuilder,
       int(const std::set<SensorId> &expected_sensor_ids,
           const proto::TrajectoryBuilderOptions &trajectory_options,
           MapBuilderInterface::LocalSlamResultCallback local_slam_result_callback));
   MOCK_METHOD1(AddTrajectoryForDeserialization,
                int(const proto::TrajectoryBuilderOptionsWithSensorIds
                            &options_with_sensor_ids_proto));
```

Directories `mapping_*d/` and namespaces `mapping_*d::` contain classes and files with the same names for 2D and 3D cases.
The code is mostly duplicated and one has to copy the same logic to both implementations for every new feature.
It makes it hard to navigate and read the code, since you never know where you are exactly without checking the path to the file.
Fixing the code duplication is a topic for a different RFC.

## Approach
[approach]: #approach
New structure:
```
common/
ground_truth/
grpc/
  internal/
  client/
  framework/
  handlers/
  proto/
  server/
io/
  internal/
mapping/
  internal/
  2d/
  3d/
sensor/
transform/
```

## Discussion Points
[discussion]: #discussion
[Internal Headers RFC](https://github.com/pifon2a/rfcs/blob/master/text/0003-internal-headers.md) talks about moving internal implementation of gRPC code to `cartographer/grpc/internal`.
Shouldn't it be `cartographer/internal/grpc`?
If no, then why do we have `cartographer/internal/mapping*` and not `cartographer/mapping*/internal`?
