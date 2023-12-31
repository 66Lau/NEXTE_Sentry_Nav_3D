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

### local planner from CMU

#### 1. loam_interface.launch
It is the interface for using custom lidar, transfer your `odom` and `PointsClouds2` to local planner standard input

- input :
  - nav_msgs::Odometry: `~Odometry_topic (default: "/Odometry" for fast_lio)`
  - sensor_msgs::PointCloud2: `~PointCloud2_topic (default: "/cloud_registered" for fast_lio)`


- ouput ：
  - sensor_msgs::PointCloud2: `/registered_scan` 
  - nav_msgs::Odometry: `/state_estimation`
  - tf tree

#### 2. sensor_scan_generation.launch
Transfer the PointCloud2 in `sensor` frame to 'map' frame

- input :
  - sensor_msgs::PointCloud2: `/registered_scan` 
  - nav_msgs::Odometry: `/state_estimation`


- ouput ：
  - sensor_msgs::PointCloud2: `/sensor_scan`
  - nav_msgs::Odometry: `/state_estimation_at_scan`
  - tf tree

#### 3. terrain_analysis.launch
analyzes the local smoothness of the terrain and associates a cost to each point on the terrain map

the terrain map is used by the collision avoidance module

- input :
  - sensor_msgs::PointCloud2: `/registered_scan` 
  - nav_msgs::Odometry: `/state_estimation`
  - tf tree
  - sensor_msgs::Joy: `/joy`
  - std_msgs::Float32: `/map_clearing`

- output : 
  - sensor_msgs::PointCloud2: `/terrain_map`

#### 4. terrain_analysis_ext.launch
 the 'terrain_analysis_ext' package extends the terrain map to a 40m x 40m area. The extended terrain map keeps lidar points over a sliding window of 10 seconds with a non-decay region within 4m from the vehicle

 extended terrain map is to be used by a high-level planning module

- input :
  - sensor_msgs::PointCloud2: `/registered_scan` 
  - sensor_msgs::PointCloud2: `/terrain_map`
  - nav_msgs::Odometry: `/state_estimation`
  - tf tree
  - sensor_msgs::Joy: `/joy`
  - std_msgs::Float32: `/map_clearing`

- output : 
  - sensor_msgs::PointCloud2: `/terrain_map_ext`


#### 5. local_planner.launch/localPlanner
Generate local path
- input :
  - sensor_msgs::PointCloud2: `/registered_scan` 
  - sensor_msgs::PointCloud2: `/terrain_map`
  - nav_msgs::Odometry: `/state_estimation`
  - tf tree
  - sensor_msgs::Joy: `/joy`
  - std_msgs::Float32: `/map_clearing`
  - geometry_msgs::PointStamped: `/way_point`
  - std_msgs::Float32: `/speed`
  - geometry_msgs::PolygonStamped: `/navigation_boundary`
  - sensor_msgs::PointCloud2: `/added_obstacles`
  - std_msgs::Bool: `/check_obstacle`

- output : 
  - nav_msgs::Path: `/path`

#### 5. local_planner.launch/pathFollower
Follow the local path, generate desired velocity
- input :
  - nav_msgs::Path: `/path`
  - sensor_msgs::Joy: `/joy`
  - std_msgs::Float32: `/speed`
  - std_msgs::int8: `/speed`
  - nav_msgs::Odometry: `/state_estimation`
- output:
  - geometry_msgs::TwistStamped: `/cmd_vel`


## Q&A

1.坡度过大是会被检测为障碍物，根据地形分析的原理，我们是将每个栅格内最低点作为地面点的，其余点相对于最低点的高度作为通过代价，坡度大的时候，单个栅格内最上面的点会比最下面的点高很多，超过可通过阈值obstacleHeightThre就会被认定为障碍物。暂时只能通过调节参数来解决，但是有可能导致正常地面可通过障碍物阈值过大，这一点需要你自己来权衡了。

2.如何判断地形分析和local_planner的设置是否正确，在rviz中选中select，然后框选想看的点云，查看intensity是否符合真实情况

3.使用不同雷达时，local_planner.launch中的pointPerPathThre非常重要，要根据自己雷达的点云密度进行修改，密度越大，这个值要适当增大
<!-- 
## bug recording

### 1. Dynamic Obstacle
problem: When some dynamic obstacles passing by, there are some pointclouds would be saved and can not be automatically clear.

-  在清理体素数组时，有一个判断条件是(laserCloudTime - systemInitTime - point.intensity <decayTime || dis < noDecayDis)我觉得这里的或应该改成和



-->