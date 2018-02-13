# Collect and Expose Metrics

## Summary

[summary]: #summary

The goal is to collect and expose structured metrics about the quality and status of the SLAM algorithm and the server that runs it.

## Motivation

[motivation]: #motivation

### Dashboard for humans

Human developers debug SLAM performance, and human operators need to monitor key metrics for the robot.
Both can profit greatly from a nice dashboard that shows key cartographer metrics, such as a histogram of constraint quality.

Today, developers often find themselves searching in log output, and the only way to monitor Cartographer is to run rviz at runtime.
We need to provide a more structured way that also works for multiple instances or longer time scales.

### Reference signal for robot control

Some manufacturers would like to have an observable for SLAM performance so that they can make downstream decisions due to bad SLAM.
Collected metrics could provide these reference signals with little intrusion in Cartographer code.

## Approach

[approach]: #approach

The approach below is illustrated by a [prototype](https://github.com/googlecartographer/cartographer/pull/876).

### Cartographer facade

Inside the cartographer library, we establish a system-agnostic facade for collecting metrics.
The facade is compatible with all important monitoring systems (Prometheus, Stackdriver, OpenCensus), but doesn't introduce build dependencies into the core cartographer lib.
The facade lives in the cartographer::metrics namespace and includes these core classes:

*   `Counter`: A single, always-increasing count of events, e.g. "number of loop iterations".
*   `Gauge`: A value that changes both up and down, e.g. "memory used" or "open requests".
*   `Histogram`: A thread-safe histogram with predefined buckets and constant memory usage.
    It's different from `cartographer::common::Histogram`, which is thread-unsafe and does the bucketing after value collection.
    Over time, I think that there would be potential to merge the two Histogram classes.
*   `CounterFamily`, `GaugeFamily`, `HistogramFamily`: An interface for obtaining `Counter`/`Gauge`/`Histogram` instances that form an aggregation domain. All metrics within a family can be safely aggregated. Different metrics in the same family can be differentiated by labels, e.g. "result" for a metric about RPC responses.
*   `Registry`: An interface that yields `HistogramFamily`, `CounterFamily` and `GaugeFamily`.
    The prototype called this `FamilyFactory`, but `Registry` should be clearer and less Prometheus-specific.

Typically, classes that support monitoring would offer a static function `RegisterMetrics` method that takes a reference to the Registry and stores references to `Counter`, `Gauge` and `Histogram` in static file-global variables.
A single function `cartographer::RegisterMetrics` calls all the individual `RegisterMetrics` functions.
To avoid race conditions, we require that `RegisterMetrics` must be called at startup time, before any multithreading is launched.

Cartographer code initializes the static file-global variables with fake instances of `Counter`, `Gauge` and `Histogram` that don't record any data.
Code that records metrics can be written agnostic of the attachment of any monitoring system.

### Implementing the facade

`cartographer_grpc` contains a concrete implementation of the facade using the [Prometheus C++ client library](https://github.com/jupp0r/prometheus-cpp).
An additional dependency to it is introduced in `cartographer_grpc` behind the optional flag `BUILD_GRPC`.
We choose [Prometheus](https://prometheus.io/) because it is a stable, well-established monitoring system for custom metrics whose network protocol is also supported by other monitoring systems (e.g.  Stackdriver).
Prometheus follows the best practices for time series data outlined in [Practical Alerting](https://landing.google.com/sre/book/chapters/practical-alerting.html).

`cartographer_grpc` opens up a monitoring endpoint on port 9100 (localhost only) where users, a Prometheus server, a sidecar running a [Prometheus-to-Stackdriver bridge](https://github.com/GoogleCloudPlatform/k8s-stackdriver/tree/master/prometheus-to-sd) or other interested parties can fetch monitoring data in a [structured
format](https://prometheus.io/docs/concepts/data_model/).

## Discussion Points

[discussion]: #discussion
