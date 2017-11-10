# Transition Cartographer From Proto2 to Proto3

## Summary
[summary]: #summary

Upgrade Cartographer Protocol Buffers dependency from proto2 to proto3 and update all `.proto` files to comply to proto3 style.

## Motivation
[motivation]: #motivation

Currently we only use Protocol Buffers for serialization purposes.
In the future however we would like to extend Cartographer to support cloud based, collaborative mapping.
This requires data exchange between individual nodes running Cartographer.
gRPC is an  open source remote procedure call system that'd be ideal for data exchange between Cartographer nodes.
gRPC however only supports proto3.
In order to avoid having to work with two Protocol Buffer versions we need to upgrade Cartographer to proto3.

## Approach
[approach]: #approach

Fortunately the proto3 library is backwards compatible in the sense that it can compile proto2 messages.
We can therefore perform the switchover in two steps:
- Upgrade the Cartographer Protocol Buffer dependency from 2 to 3.
- Refactor any code that uses proto2 features ('has...') and convert (at the very least) those messages that we intend to send over the wire to proto3 style.

This [PR][FirstPR] summarizes the plan for step 1 for the Cartographer repo:
- We add a script 'install_proto3.sh' which clones proto3 from HEAD, compiles and installs it to '/usr/local'.
- This script is used in our Dockerfiles to install proto3 and replaces our former 'apt install protobuf-compiler' which installed proto2.
- We update the build instructions for Cartographer to include proto3 as a necessary prerequisite.

A similar PR will be written for the 'cartographer_ros' repo.

## Discussion Points
[discussion]: #discussion

[FirstPR]: https://github.com/googlecartographer/cartographer/pull/644
