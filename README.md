# Navigation system in 3D environment(for real robot)

## Overview
Under development, waiting for improvement

## Environment Setting

Hardware:
- minipc i7-8550U
- RAM 32G
- MID360(Install upside down)


Environment:
- ROS noetic
- pre-requirement setting 
  - [Fast-lio](https://github.com/hku-mars/FAST_LIO)
  - [CMU-autonomous_exploration_development_environment](https://github.com/HongbiaoZ/autonomous_exploration_development_environment)
  - Move-Base


## Framework

 1. Publishing `2D Nav goal` using `rviz` or `sentry_decision`
 2. Generating `global path` using `move_base`
 3. Converting `global path` to a series of `waypoint` by using `path2waypoint` node
 4. Generating `local path` and `cmd_vel` using `CMU_local_planner`
 5. Using `sentry_serial` to impart `cmd_vel` with the real robot driver

## In&out put

### cmu base planner

input :
  - sensor_msgs::PointCloud2: `/registered_scan` 
  - nav_msgs::Odometry: `/state_estimation`
  - tf tree

  TBD...

<!-- 
## bug recording

### 1. Dynamic Obstacle
problem: When some dynamic obstacles passing by, there are some pointclouds would be saved and can not be automatically clear.

-  在清理体素数组时，有一个判断条件是(laserCloudTime - systemInitTime - point.intensity <decayTime || dis < noDecayDis)我觉得这里的或应该改成和


-->