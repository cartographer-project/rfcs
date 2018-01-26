# Struct sensor_id

## Summary
[summary]: #summary

Replace the `string` sensor_id by a `struct` which allows us to include sensor metadata, such as an `enum SensorType`.

## Motivation
[motivation]: #motivation

When distributing local and global slam over different processes, identifying sensors by a string is no longer sufficient, because multiple range finder sensors in local SLAM become a single LocalSlamResult sensor on the global side.
For example, when local SLAM runs on an agent server and global SLAM on an uplink server, the agent server expects `n` range finder sensors and additional sensors, while the collator of the uplink server waits for only one "LocalSlamResultData" input and the addtional sensors.

## Approach
[approach]: #approach

The suggested change is to replace `std::string sensor_id` by a struct that contains an `enum SensorType` and a `std::string id`.
This allows us to implement distributed local/global mapping in a clean way, such as removing range finder sensors from the set and adding a sensor type for local slam results.

As an additional benefit, future concepts about properties of sensors would be easy to implement.
For instance, it would be easy to include collator timeouts or tag "sparse" sensors that processing should not wait for.

## Discussion Points
[discussion]: #discussion

Alternatives would be:

* Passing two separate sets of `expected_sensor_ids` to `AddTrajectoryBuilder`, the new one `expected_rangefinder_sensor_ids`.
* Matching well-known strings like "scan" or "rangefinder" and removing them from a set.


