# Serialization Format

## Summary

[summary]: #summary

Definition of a versioned, backwards-compatible and extensible serialization
format for the state of Cartographer.

This RFC is a small deviation of the previously proposed header format from
[RFC-0015](https://github.com/googlecartographer/rfcs/blob/master/text/0015-serialization-header.md).

#### Note: This is **NOT** a definition of the binary file format on disk.

This RFC is mainly addressing the logical order of protobuf messages within a
decoded binary stream (where decoded e.g. refers to the decompressed message
stream for *pbstream* files).

## Motivation

[motivation]: #motivation

At the moment, the file format for serializing the state of Cartographer is
defined purely within the serialization & deserialization routines of
Cartographer. Since we serialize all information into a binary stream of
`protobuf` messages (*pbstream*) without a format-version information, extending
or changing the format inevitably leads to a breaking change for Cartographer
users, as there is no way of falling back to load previously serialized states.

This RFC is going to address this issue by introducing a `SerializationHeader`
message and an interface for parsing serialized states, independent of the
serialization format.

## Approach

[approach]: #approach

For being able to identify the underlying serialization format, we introduce a
`SerializationHeader` message.

```
message SerializationHeader {
  uint32 format_version = 1;
}
```

We further change the current `SerializedData` protobuf message, which is meant
to be able to contain `oneof` any of the message types required for serializing
the mapping state:

```
message SerializedData {
  oneof data {
    PoseGraph pose_graph = 1;
    AllTrajectoryBuilderOptions all_trajectory_builder_options = 2;
    Submap submap = 3;
    TrajectoryData trajectory_data = 4;
    Node node = 5;
    ImuData imu_data = 6;
    OdometryData odometry_data =7;
    FixedFramePoseData fixed_frame_pose_data = 8;
    LandmarkData landmark_data = 9;
  }
}
```

~~For backwards compatibility, we will rename the original `SerializedData`
definition to `LegacySerializedData`.~~ (see discussion)

By keeping all the payload related data wrapped in a single message type, we can
also easily switch to a different underlying storage format in the future (e.g.
[Riegeli](https://github.com/google/riegeli/)).

### Generic Serialization Order

Using the above messages, the generic serialization order is defined as:

*Generic Serialization Order* |
:---------------------------: |
SerializationHeader           |
(SerializedData)*             |

### Version 1.0 Serialization Order

Using the definitions above, the proposed serialization format for version 1.0
is as follows

*Version 1.0 Serialization Order* |
:-------------------------------: |
SerializationHeader               |
PoseGraph                         |
AllTrajectoryBuilderOptions       |
(Submap)\*                        |
(Node)\*                          |
(ImuData)\*                       |
(OdometryData)\*                  |
(FixedFramePoseData)\*            |
(TrajectoryData)\*                |
(LandmarkData)\*                  |

### Backwards-Compatibility

*   ~~To gracefully transition to the new format, we will provide an internal
    fallback to the *old* parsing of PbStreams. Meaning, we will first try to
    read the a `SerializationHeader` from the pbstream and if this fails, try
    parsing the *old* format. This way we can at least provide functionality for
    parsing the last pre-header version.~~ (see discussion)
*   We will provide a migration tool to re-write already serialized pbstreams
    into the new format.

## Discussion Points

[discussion]: #discussion

*   Version string for new format: 0.9 or 1.0?
    -   `uint32` - first version == 1

Discussion during Open-House:

*   SirVer: Cartographer is pre 1.0, why do we want to support the old file
    format. Migration tool sounds sufficient.
*   **Decision**: Only provide the proposed migration tool and do not fallback
    to try loading mapping states using old deserialization routines.
