# Pose Graph Refactoring 1: Pose Graph Data
## Summary
[summary]: #summary

Move the data that models pose graph in 2D or 3D to `PoseGraphData` struct.

## Motivation
[motivation]: #motivation

PoseGraph2D and PoseGraph3D are [God objects](https://en.wikipedia.org/wiki/God_object) with a lot of code duplication. They can be refactored and split.
 Although there is some logic/data specific to 2D/3D case, some parts are dimension agnostic and can be unified. 

Data members of  `cartographer::mapping::PoseGraph2D` ([link](https://github.com/googlecartographer/cartographer/blob/master/cartographer/mapping/internal/2d/pose_graph_2d.h)).

```cpp
// Classes/data that should belong to the 'PoseGraphController'.
ConstraintBuilder2D constraint_builder_;
GlobalSlamOptimizationCallback global_slam_optimization_callback_;
bool run_loop_closure_ = false;
const proto::PoseGraphOptions options_;
int num_nodes_since_last_loop_closure_;
mutable common::Mutex mutex_;
std::unique_ptr<OptimizationProblem2D> optimization_problem_;
std::unique_ptr<std::deque<std::function<void()>>> work_queue_;
std::unordered_map<int, std::unique_ptr<FixedRatioSampler>>
    global_localization_samplers_;
std::vector<std::unique_ptr<PoseGraphTrimmer>> trimmers_;

// Actual state of the pose graph.
MapById<NodeId, TrajectoryNode> trajectory_nodes_;
MapById<SubmapId, InternalSubmapData2D> submap_data_;
MapById<SubmapId, SubmapSpec2D> global_submap_poses_;
TrajectoryConnectivityState trajectory_connectivity_state_;
int num_trajectory_nodes_;
std::map<int, InitialTrajectoryPose> initial_trajectory_poses_;
std::map<std::string, PoseGraph::LandmarkNode> landmark_nodes_;
std::set<int> finished_trajectories_;
std::set<int> frozen_trajectories_;
std::vector<Constraint> constraints_;
```

Data members of  `cartographer::mapping::PoseGraph3D` ([link](https://github.com/googlecartographer/cartographer/blob/master/cartographer/mapping/internal/3d/pose_graph_3d.h)).

```cpp
// Classes/data that should belong to the 'PoseGraphController'.
ConstraintBuilder3D constraint_builder_;
GlobalSlamOptimizationCallback global_slam_optimization_callback_;
bool run_loop_closure_ = false;
const proto::PoseGraphOptions options_;
int num_nodes_since_last_loop_closure_;
mutable common::Mutex mutex_;
std::unique_ptr<OptimizationProblem3D> optimization_problem_;
std::unique_ptr<std::deque<std::function<void()>>> work_queue_;
std::unordered_map<int, std::unique_ptr<FixedRatioSampler>>
    global_localization_samplers_;
std::vector<std::unique_ptr<PoseGraphTrimmer>> trimmers_;

// Actual state of the pose graph.
MapById<NodeId, TrajectoryNode> trajectory_nodes_;
MapById<SubmapId, InternalSubmapData3D> submap_data_;
MapById<SubmapId, SubmapSpec3D> global_submap_poses_;
TrajectoryConnectivityState trajectory_connectivity_state_;
int num_trajectory_nodes_;
std::map<int, InitialTrajectoryPose> initial_trajectory_poses_;
std::map<std::string, PoseGraph::LandmarkNode> landmark_nodes_;
std::set<int> finished_trajectories_;
std::set<int> frozen_trajectories_;
std::vector<Constraint> constraints_;
```

This RFC is about unifying the members that belong to the *'Actual state of the pose graph'* section. 

Most of the data types are identical except for the ones that have `2D` or `3D` suffix explicitly.
The remaining members are 

*  `MapById<SubmapId, SubmapSpec2D> global_submap_poses_` vs `MapById<SubmapId, SubmapSpec3D> global_submap_poses_`
*  `MapById<SubmapId, InternalSubmapData2D> submap_data_` vs `MapById<SubmapId, InternalSubmapData3D> submap_data_`

Note, that `InternalSubmapData2D` and `InternalSubmapData3D` structs differ only in the derived classes they point to and can be merged.
```cpp
struct InternalSubmapData2D {
 std::shared_ptr<const Submap2D> submap;
 std::set<NodeId> node_ids;
 SubmapState state = SubmapState::kActive;
};

struct InternalSubmapData3D {
 std::shared_ptr<const Submap3D> submap;
 std::set<NodeId> node_ids;
 SubmapState state = SubmapState::kActive;
};
```

## Approach
[approach]: #approach
Introduce `PoseGraphData` and reuse it in `PoseGraph2D` and `PoseGraph3D` classes.

```cpp
struct PoseGraphData {
  MapById<SubmapId, InternalSubmapData> submap_data;
  MapById<SubmapId, SubmapSpec2D> global_submap_poses_2d;
  MapById<SubmapId, SubmapSpec3D> global_submap_poses_3d;

  MapById<NodeId, TrajectoryNode> trajectory_nodes;
  TrajectoryConnectivityState trajectory_connectivity_state;
  int num_trajectory_nodes;
  std::map<int, InitialTrajectoryPose> initial_trajectory_poses;
  std::map<std::string, PoseGraph::LandmarkNode> landmark_nodes;
  std::set<int> finished_trajectories;
  std::set<int> frozen_trajectories;
  std::vector<Constraint> constraints;
};
```

## Discussion Points
[discussion]: #discussion

What is the preferred name: `PoseGraphModel` or `PoseGraphData` or `PoseGraphState`?
