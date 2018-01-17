# Improved handling of multiple trajectories and bags in the offline node

## Summary
[summary]: #summary

The offline node currently supports multiple sequential (non-concurrent) trajectories via loading multiple bags, where each bag is a separate trajectory.
This RFC aims to give the user greater flexibility with the offline node when dealing with multiple robots, multiple (possibly concurrent) trajectories and multiple bags.

## Motivation
[motivation]: #motivation

The online node currently supports tracking multiple concurrent trajectories in real time.
This enables the Cartographer user to do cooperative SLAM using multiple robots.
However, support for this in the offline node is currently limited - only a single robot configuration may be used, and the trajectories cannot be concurrent.
Since this functionality is supported in the online node, this RFC aims to expand the offline node to be able to provide the same functionality, but at increased speed. 

For example, a user has bagged a swarm of robots and wants to try out cooperative SLAM with Cartographer.
They are interested in crunching the bags as fast as possible.
Perhaps they have a bag for each robot, perhaps they have all robots in a single bag - the offline node should have a flexible configuration interface to allow for various possibilities.

The concurrent trajectories could be processed either sequentially or in parallel - it should not influence the end result in any way, since local SLAM is not influenced by other trajectories, and global SLAM should arrive at the same pose graph at the end, no matter what the order of insertion is.
However, processing and displaying the concurrent trajectories in parallel, like the online node can do, would be more useful and increase the coolness factor.


### Overview of possible dataset scenarios
Let's give an overview of all possibilities how a dataset layout and content could look like:

| Robot plurality                                                         | Trajectory plurality                                 | Bag plurality                                                          |
|-------------------------------------------------------------------------|------------------------------------------------------|------------------------------------------------------------------------|
| __A.__ Single robot                                                     | __1.__ Single trajectory                             | __a.__ Single bag                                                      |
| __B.__ Multiple identical robots  (using the same sensor topics/frames) | __2.__ Multiple trajectories, necessarily sequential | __b.__ Multiple continuous bags  (like a single bag) - a bag aggregate |
| __C.__ Multiple different robots  (different sensor topics/frames)      | __3.__ Multiple trajectories,  possibly concurrent   | __c.__ Multiple bags                                                   |

The online node currently supports the following combinations:
__A1__, __A/B 2__ and __C 2/3__ (using `StartTrajectory` and `FinalizeTrajectory` service calls)

The offline node currently supports: __A1a__, __A/B 2c__.

The online node can never support __B3__, because only single robot's data can be published on a topic at a time.
Also, __B3__ and __a/b__ is impossible for the same reasons.

Note that some combinations are invalid, such as __A3__ (a single robot can't be on multiple concurrent trajectories), and __B/C 1__ (there can't be a single trajectory if there are multiple robots).
Furthermore, __A/B 2 a/b__ would require us to be able to specify when a trajectory stops and begins within the same bag aggregate and on the same sensor data topics, which would be too complicated and out of scope.

The aim of the RFC is to enable the user to use the offline node for __C2/3__, like the online node; to preserve the ability of doing __A/B 2c__; and, to possibly offer additional functionality. For example, __b__ (bag aggregates), and __B3c__, which would otherwise be impossible with the online node.

## Approach
[approach]: #approach

The proposed offline node configuration for multiple, possibly concurrent trajectories (__3__) should closely follow the way of the online node for achieving this, listed here for reference:
  - Performing multiple calls of the `StartTrajectory` service, with possibly different `TrajectoryOptions` and necessarily different sensor data topics (__C__); all is specified in the service request.
  - Alternatively, the `start_trajectory` helper node is invoked multiple times, with possibly different `.lua` files for `TrajectoryOptions` and necessarily different sensor data topics (__C__); the recently merged PR googlecartographer/cartographer_ros#584 enables specifying different topics via topic remapping.
  - Specially, if a trajectory is finalized, we may start another independent trajectory which uses the same sensor data topics and possibly different `TrajectoryOptions` (__B2__).
  This is already achievable in the offline node by simply loading multiple bags, which creates a trajectory for each bag, although `TrajectoryOptions` are necessarily the same for all trajectories (__B2c__). 
  The ability to do this has to be preserved or expanded.

If we suppose that there is a mandated 1:1 relationship between bags and trajectories in the offline node, the following configuration can be proposed.

### Configuration #1

__For each bag (trajectory)__, the configuration is a __tuple__ consisting of:
  - `TrajectoryOptions` (given in a `.lua` file)
  - A set of sensor data topics
  - Boolean `use_bag_transforms` which allows us to ignore transforms in the bag
  - An optional `.urdf` file.

Both `TrajectoryOptions` and the sensor data topics can be identical for multiple different bags (preserving __B2c__). However, in addition to that, the trajectories can be concurrent (__B3c__).

Because the trajectories may be concurrent, due care would need to be taken to ensure that data published on the same topic, but in different bags, gets routed to the appropriate trajectory.

### Additional considered variants

Going further, allowing multiple trajectories in a single bag (__C 2/3 a__) and relaxing the 1 trajectory : 1 bag restriction would mean that the configuration of each bag becomes a set of tuples mentioned above.
This is given in Configuration #2.

#### Configuration #2

Similar to Configuration #1, but have a __set of tuples for each bag__.
Each tuple describes a trajectory in the bag.

For both Configuration #1 and Configuration #2, all concurrent trajectories (whether in the same bag as permitted by Configuration #2, or in different bags) are __collated__.

#### Handling split bags

Another common use case is (__b__) - a single continuous trajectory is split into multiple bags (e.g. when using the split option of `rosbag record`).

This could be handled by defining a `BagAggregate`: a list of one or more continuous bags, which are to be treated as one bag.

#### Configuration #3
Configuration is given as a __set of tuples for each `BagAggregate`__, each tuple describing a trajectory in the `BagAggregate`.

As before, any concurrent trajectories are collated.
Each `BagAggregate` has its own transform buffer (which is shared by all bags in the aggregate).

### Decision

Configuration #1 is to be used and implemented. The case covered by Configuration #2 is covered by specifying the same bag multiple times, but with different sensor topics. A single TF buffer is used.

### Data format of the configuration

The comma-separated list approach, as already used for `bag_filenames`, has been expanded to `configuration_basenames`, `urdf_filenames`, and there is now `sensor_topics` as well. The tuples from Configuration #1 are thus specified positionally in the aforementioned command line arguments - i.e. the first comma-delimited item in each command line argument corresponds to the first tuple, and so on.

Here is an example of launching the offline node for two robots named alpha and delta, which were driven simultaneously, and their sensor data being captured in two corresponding bags:
```
rosrun cartographer_ros cartographer_offline_node
-configuration_directory /somewhere
-configuration_basenames alpha.lua,delta.lua -urdf_filenames alpha.urdf,delta.urdf
-bag_filenames alpha.bag,delta.bag -sensor_topics alpha/scan:alpha/pose,delta/scan:delta/pose
```

The sensor topics for different trajectories are separated by commas. The topics for each trajectory are further separated with colons. Leaving sensor topics unspecified causes the default computed topic names to be used.

If only one configuration basename or set of topics is specified, it will be used for all trajectories, thus preserving the existing behaviour.

## Discussion Points
[discussion]: #discussion

Should Configuration #1, #2 or #3 be used?

Answer: Configuration #1.
