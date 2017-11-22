# Internal Header Files

## Summary
[summary]: #summary

To be able to eventually release a version 1.0.0, Cartographer needs a public API.
One part of this is a C++ API which consists of header files intended to be used by the users of libcartographer.
Other header files are purely internal and should not be made available to the users.

## Motivation
[motivation]: #motivation

We currently install all header files contained in the libcartographer code base which exposes many implementation details.
Currently there is no stability guarantee for the API and header files can change in ways that break users.
We want to have a clear separation between internal and public header files.
Only the latter are to be installed with libcartographer.

## Approach
[approach]: #approach

We would like to be able to put a Bazel `WORKSPACE` file in the repository root to possibly build Cartographer with Bazel in the future.
[Bazel best practices for C++](https://docs.bazel.build/versions/master/bazel-and-cpp.html#include-paths) suggest making all include paths relative to the workspace directory and using quoted includes.
We also want to include public header files as `#include "cartographer/foo.h"`.
This is currently done and we would like to retain this.

Moreover we would like to install only some of the header files which form the public C++ API.
We will move some of the files into a directory structure beneath `cartographer/internal`.
Header files in `cartographer/internal` will not be installed.
No public header file can include these internal headers.
Similarly if gRPC-related code for cloud-based mapping is put inside a `cartographer/grpc` directory, its internal implementation will be put inside `cartographer/grpc/internal`.

## Discussion Points
[discussion]: #discussion
