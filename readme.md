# Differential-Drive Mobile Robot — ROS 2 Jazzy & Gazebo Harmonic

A modular differential-drive mobile robot developed in **ROS 2 Jazzy** and **Gazebo Harmonic**, with a Xacro/URDF-based robot model, simulated LiDAR, differential-drive control, odometry, TF, joint-state publishing, and ROS 2 ↔ Gazebo communication through `ros_gz_bridge`.

The project is structured as a software foundation for progressively moving from robot modelling and manual teleoperation toward **RViz visualization, SLAM, autonomous navigation, and eventual physical hardware deployment**.

---

## 1. Project Overview

The robot is a **two-wheel differential-drive mobile platform** supported by a caster. The left and right drive wheels are independently actuated, while the caster provides passive support.

The current software stack is:

```text
                    ROS 2 Jazzy
                        │
        ┌───────────────┼────────────────┐
        │               │                │
   /cmd_vel          /joint_states      TF
        │               │                │
        ▼               ▼                ▼
   DiffDrive      Joint State      Robot State
    Control        Publisher       Publisher
        │
        ▼
                 Gazebo Harmonic
                        │
                 ┌──────┴──────┐
                 │             │
              Physics        LiDAR
                 │             │
               /odom         /scan
                 │             │
                 └──────┬──────┘
                        ▼
                 ros_gz_bridge
                        │
                        ▼
                    ROS 2 Topics
```

### Current capabilities

- Differential-drive mobile robot model
- Xacro/URDF robot description
- Continuous wheel joints
- Fixed caster and LiDAR mounting
- Gazebo Harmonic physics and simulation
- Gazebo DiffDrive system
- Keyboard teleoperation through `/cmd_vel`
- Odometry publishing through `/odom`
- TF publishing
- Joint-state publishing
- Simulated GPU LiDAR
- `sensor_msgs/msg/LaserScan` bridge on `/scan`
- ROS 2 ↔ Gazebo communication using `ros_gz_bridge`
- Package configuration using `package.xml` and CMake/installation rules

### Planned extensions

```text
Current
  ↓
LiDAR + /scan
  ↓
RViz visualization
  ↓
SLAM
  ↓
Map generation
  ↓
Nav2 / autonomous navigation
  ↓
Physical hardware
```

---

# 2. Robot Architecture

## 2.1 Differential Drive

The robot uses two independently driven wheels:

```text
                 Front

          Left wheel     Right wheel
              O               O
               \             /
                \   BODY    /
                 \_________/
                      |
                    Caster
```

The two wheel joints are:

```text
wheel1_joint
wheel2_joint
```

Both are defined as `continuous` joints and rotate around:

```xml
<axis xyz="0 1 0"/>
```

The differential-drive controller converts the commanded robot velocity into individual wheel velocities.

For a robot with wheel radius `r` and wheel separation `L`:

\[
v = \frac{r}{2}(\omega_R+\omega_L)
\]

\[
\omega = \frac{r}{L}(\omega_R-\omega_L)
\]

where:

- `v` = linear velocity
- `ω` = angular velocity
- `ωR` = right-wheel angular velocity
- `ωL` = left-wheel angular velocity
- `r` = wheel radius
- `L` = distance between the driven wheels

### Basic motion

| Command | Wheel behaviour | Result |
|---|---|---|
| Equal positive velocities | Both wheels forward | Move forward |
| Equal negative velocities | Both wheels backward | Move backward |
| Left wheel slower than right | Different wheel speeds | Turn left |
| Right wheel slower than left | Different wheel speeds | Turn right |
| Opposite wheel velocities | Wheels rotate in opposite directions | Rotate in place |

---

# 3. Degrees of Freedom

For planar motion, the mobile robot has **3 degrees of freedom**:

```text
x   → translation along X
y   → translation along Y
θ   → rotation about Z (yaw)
```

Therefore:

> **Planar configuration DOF = 3**

This should not be confused with actuator count. The robot has **two independently driven wheel joints**, which provide the differential-drive actuation.

---

# 4. Robot Description — `robot.xacro`

File:

```text
/home/arshit/ws_mobile/src/mobile_robot/model/robot.xacro
```

`robot.xacro` is the main robot description file. Xacro is used instead of writing a large URDF directly because it allows reusable parameters, expressions and macros.

The model contains the major components:

```text
base_footprint
      │
      ▼
  body_link
   ├───────┐
   ▼       ▼
wheel1   wheel2
   │       │
   └───┬───┘
       ▼
     caster

       +
       │
       ▼
   lidar_link
```

### Main links

- `base_footprint`
- `body_link`
- `wheel1_link`
- `wheel2_link`
- `caster_link`
- `lidar_link`

### Main joints

- `base_footprint` fixed relationship to the body
- `wheel1_joint` — continuous
- `wheel2_joint` — continuous
- caster joint — fixed
- `lidar_joint` — fixed

### Wheel configuration

The driven wheels are cylindrical links. Their visual and collision geometry are oriented so that the wheel axle corresponds to the configured joint axis.

The wheel joints include effort/velocity limits and damping/friction parameters so that the simulated robot has physically meaningful joint behaviour.

---

# 5. LiDAR Sensor

The robot contains a simulated GPU LiDAR attached through:

```xml
<joint name="lidar_joint" type="fixed">
```

and:

```xml
<link name="lidar_link">
```

The Gazebo sensor is defined as:

```xml
<sensor name="gpu_lidar" type="gpu_lidar">
```

The configured sensor publishes:

```text
/scan
```

The sensor configuration includes a sample count of:

```text
720 samples
```

Each scan provides a set of range measurements across the configured angular field of view.

Conceptually:

```text
Laser ray
    │
    ▼
Obstacle
    │
    ▼
Distance measurement
    │
    ▼
LaserScan message
```

The ROS 2 message contains data such as:

```text
angle_min
angle_max
angle_increment
range_min
range_max
ranges[]
```

The `ranges[]` array contains the measured distances for the individual angular samples.

### Important distinction

LiDAR provides **perception data**. It does not, by itself, perform obstacle avoidance.

A future navigation or obstacle-avoidance node would consume `/scan` and generate an appropriate `/cmd_vel`.

---

# 6. Gazebo Configuration — `robot.gazebo`

File:

```text
/home/arshit/ws_mobile/src/mobile_robot/model/robot.gazebo
```

This file contains Gazebo-specific configuration that complements the Xacro/URDF robot description.

## 6.1 Material and friction configuration

Gazebo-specific properties are assigned to the robot components, including:

- Body material
- Wheel material
- Caster material
- Friction coefficients

The drive wheels use different friction settings from the caster so that the caster can behave as a passive supporting element.

---

## 6.2 Differential-Drive Plugin

Gazebo uses:

```xml
<plugin
    filename="gz-sim-diff-drive-system"
    name="gz::sim::systems::DiffDrive">
```

The plugin is configured with the two wheel joints:

```xml
<right_joint>wheel1_joint</right_joint>
<left_joint>wheel2_joint</left_joint>
```

It also defines:

- wheel separation
- wheel diameter
- maximum linear acceleration
- command velocity topic
- odometry topic
- TF topic
- parent/child frames
- odometry publication frequency

Key topics:

```text
cmd_vel
odom
tf
```

---

# 7. Joint-State Publisher

Gazebo also uses:

```xml
<plugin
    filename="gz-sim-joint-state-publisher-system"
    name="gz::sim::systems::JointStatePublisher">
```

The wheel joints are included:

```xml
<joint_name>wheel1_joint</joint_name>
<joint_name>wheel2_joint</joint_name>
```

This provides wheel joint state information for ROS 2.

---

# 8. Gazebo LiDAR Configuration

The LiDAR is attached to:

```xml
<gazebo reference="lidar_link">
```

and configured using:

```xml
<sensor name="gpu_lidar" type="gpu_lidar">
```

The sensor publishes:

```xml
<topic>scan</topic>
```

Gazebo therefore exposes the sensor stream as:

```text
/scan
```

The scan is then bridged to ROS 2 through `ros_gz_bridge`.

---

# 9. ROS 2 ↔ Gazebo Bridge

File:

```text
/home/arshit/ws_mobile/src/mobile_robot/model/bridge_parameters.yaml
```

The bridge configuration maps Gazebo Transport messages to ROS 2 message types.

## Bridge directions

```text
GZ_TO_ROS
```

means:

```text
Gazebo → ROS 2
```

while:

```text
ROS_TO_GZ
```

means:

```text
ROS 2 → Gazebo
```

### Current communication mapping

| Purpose | ROS 2 topic | Gazebo topic | Direction |
|---|---|---|---|
| Simulation clock | `/clock` | `clock` | GZ → ROS |
| Joint states | `/joint_states` | `joint_states` | GZ → ROS |
| Odometry | `/odom` | `odom` | GZ → ROS |
| TF | `/tf` | `tf` | GZ → ROS |
| Velocity command | `/cmd_vel` | `cmd_vel` | ROS → GZ |
| LiDAR | `/scan` | `scan` | GZ → ROS |

The LiDAR mapping is:

```yaml
- ros_topic_name: "scan"
  gz_topic_name: "scan"
  ros_type_name: "sensor_msgs/msg/LaserScan"
  gz_type_name: "gz.msgs.LaserScan"
  direction: GZ_TO_ROS
  lazy: false
```

`lazy: false` keeps the bridge active rather than only forwarding data when a ROS subscriber is present.

---

# 10. ROS 2 Topics

The main runtime topics are:

### `/cmd_vel`

Message:

```text
geometry_msgs/msg/Twist
```

Used to command linear and angular velocity.

Important fields:

```text
linear.x
angular.z
```

---

### `/scan`

Message:

```text
sensor_msgs/msg/LaserScan
```

Contains LiDAR range measurements.

---

### `/odom`

Message:

```text
nav_msgs/msg/Odometry
```

Contains robot odometry information.

---

### `/joint_states`

Message:

```text
sensor_msgs/msg/JointState
```

Contains state information for robot joints.

---

### `/tf`

Message:

```text
tf2_msgs/msg/TFMessage
```

Carries coordinate-frame transformations.

---

### `/clock`

Message:

```text
rosgraph_msgs/msg/Clock
```

Provides Gazebo simulation time to ROS 2.

---

# 11. TF and Coordinate Frames

The project uses TF to maintain the relationships between robot coordinate frames.

A simplified frame tree is:

```text
odom
  │
  ▼
base_footprint
  │
  ▼
body_link
  ├── wheel1_link
  ├── wheel2_link
  ├── caster_link
  └── lidar_link
```

TF is critical for sensor visualization and future SLAM/navigation because ROS must know where the LiDAR is located relative to the robot.

For example:

```text
lidar_link
    ↓
Where is the LiDAR measurement relative to
base_footprint?
```

TF provides that transformation.

---

# 12. Keyboard Teleoperation

The project can be controlled with the ROS 2 `teleop_twist_keyboard` package.

Start it with:

```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

Common controls:

```text
        U   I   O
        J   K   L
        M   ,   .
```

Typical behaviour:

```text
I → forward
, → backward
J → rotate left
L → rotate right
K → stop
U → forward + left
O → forward + right
M → backward + left
. → backward + right
```

The keyboard node publishes `Twist` commands on:

```text
/cmd_vel
```

Those commands reach Gazebo through `ros_gz_bridge`, where the DiffDrive system converts them into left and right wheel motion.

---

# 13. Package Metadata — `package.xml`

File:

```text
/home/arshit/ws_mobile/src/mobile_robot/model/package.xml
```

The package metadata declares the ROS 2 package name, build type, license/maintainer metadata and runtime/build dependencies.

The relevant dependencies for this project include:

```xml
<depend>joint_state_publisher</depend>
<depend>robot_state_publisher</depend>
<exec_depend>xacro</exec_depend>
<depend>ros_gz</depend>
<depend>ros_gz_sim</depend>
<depend>ros_gz_bridge</depend>
```

These dependencies provide the components required for:

- Robot state publishing
- Joint state publishing
- Xacro processing
- Gazebo integration
- Gazebo simulation
- ROS 2 ↔ Gazebo message bridging

---

# 14. Build Configuration — `CMakeLists.txt`

File:

```text
/home/arshit/ws_mobile/src/mobile_robot/model/CMakeLists.txt
```

The CMake configuration is responsible for building the ROS 2 package and installing runtime resources such as launch, model and parameter files.

The project uses the ROS 2 CMake/ament build system.

A typical resource installation rule used in the project is:

```cmake
install(
  DIRECTORY launch model parameters
  DESTINATION share/${PROJECT_NAME}
)
```

The purpose of this rule is to ensure that runtime resources remain available from the package's installed share directory after:

```bash
colcon build
```

---

# 15. Installation and Environment Setup

The development environment uses:

```text
Ubuntu
ROS 2 Jazzy
Gazebo Harmonic
```

Source the ROS 2 environment:

```bash
source /opt/ros/jazzy/setup.bash
```

After building the workspace, source the local workspace:

```bash
source ~/ws_mobile/install/setup.bash
```

It is good practice to source both before working with the package in a new terminal.

---

# 16. Workspace Structure

The overall ROS 2 workspace is:

```text
~/ws_mobile/
├── src/
│   └── mobile_robot/
│       ├── CMakeLists.txt
│       ├── package.xml
│       ├── launch/
│       ├── model/
│       │   ├── robot.xacro
│       │   ├── robot.gazebo
│       │   ├── bridge_parameters.yaml
│       │   ├── CMakeLists.txt
│       │   └── package.xml
│       └── parameters/
│
├── build/
├── install/
└── log/
```

> Keep the package layout consistent with the actual CMake/package configuration. ROS 2 launch and runtime resources must be installed into the package's share directory so that `ros2 launch` can locate them after building.

---

# 17. Build the Workspace

From the workspace root:

```bash
cd ~/ws_mobile
colcon build
```

After the build completes:

```bash
source /opt/ros/jazzy/setup.bash
source ~/ws_mobile/install/setup.bash
```

---

# 18. Launching the Robot

The project's Gazebo launch file starts the required Gazebo and ROS 2 processes.

The launch workflow is conceptually:

```text
Launch
  │
  ├── Gazebo Harmonic
  │
  ├── Robot State Publisher
  │
  ├── Spawn robot from robot_description
  │
  └── ros_gz_bridge
```

Launch using:

```bash
ros2 launch mobile_robot gazebo_model.launch.py
```

---

# 19. Verifying the System

## Check ROS 2 topics

```bash
ros2 topic list
```

Check LiDAR:

```bash
ros2 topic list | grep scan
```

Check the LiDAR message type:

```bash
ros2 topic type /scan
```

Expected:

```text
sensor_msgs/msg/LaserScan
```

Inspect one LiDAR message:

```bash
ros2 topic echo /scan --once
```

Check Gazebo topics:

```bash
gz topic -l
```

Search for LiDAR-related topics:

```bash
gz topic -l | grep -Ei "scan|lidar"
```

The Gazebo LiDAR stream should expose:

```text
/scan
```

---

# 20. RViz Integration

RViz is the visualization layer for ROS 2 data.

A typical visualization setup is:

```text
Gazebo
   │
   ├── /scan
   ├── /odom
   └── /tf
         │
         ▼
        RViz
```

For LiDAR:

1. Start `rviz2`.
2. Set the **Fixed Frame** to a valid robot frame such as `base_footprint`.
3. Add a **LaserScan** display.
4. Set its topic to:

```text
/scan
```

The LiDAR measurements should then appear around the robot.

---

# 21. What the LiDAR Pipeline Does

The complete sensor pipeline is:

```text
Environment
    │
    ▼
Gazebo geometry
    │
    ▼
GPU LiDAR ray casting
    │
    ▼
Gazebo `gz.msgs.LaserScan`
    │
    ▼
ros_gz_bridge
    │
    ▼
ROS 2 `sensor_msgs/msg/LaserScan`
    │
    ▼
/scan
    │
    ├── RViz
    └── Future SLAM / Navigation
```

---

# 22. Future SLAM Pipeline

The project is structured to support SLAM.

SLAM means:

> **Simultaneous Localization and Mapping**

The future pipeline is:

```text
LiDAR
  │
  ▼
/scan
  +
Odometry
  +
TF
  │
  ▼
SLAM
  │
  ├── Robot pose
  └── Occupancy map
```

SLAM will allow the robot to estimate its position while constructing a map of the environment.

---

# 23. Future Autonomous Navigation

Once SLAM and mapping are available, the project can be extended toward autonomous navigation:

```text
                  Map
                   │
                   ▼
                  Nav2
                   │
         ┌─────────┴─────────┐
         │                   │
     Localization        Path Planning
         │                   │
         └─────────┬─────────┘
                   ▼
                /cmd_vel
                   │
                   ▼
             Differential Drive
                   │
                   ▼
                 Robot
```

This would allow the operator to provide a goal instead of manually driving the robot.

---

# 24. Transition to Physical Hardware

The long-term objective is to transfer the software architecture from Gazebo to a physical differential-drive platform.

The hardware version would replace simulated components with:

```text
Simulated LiDAR  → Physical LiDAR
Gazebo motors    → Motor driver + motors
Gazebo odometry  → Encoder-based odometry
Gazebo IMU       → Physical IMU
Gazebo physics   → Real robot dynamics
```

The ROS 2 interfaces can remain conceptually similar, allowing the high-level software stack to remain largely unchanged.

---

# 25. Software Architecture Summary

The complete current project can be summarized as:

```text
                   ┌─────────────────────┐
                   │      Teleop         │
                   │ teleop_twist_keyboard│
                   └──────────┬──────────┘
                              │
                           /cmd_vel
                              │
                              ▼
                     ┌─────────────────┐
                     │ ros_gz_bridge   │
                     └────────┬────────┘
                              │
                              ▼
                     ┌─────────────────┐
                     │ Gazebo DiffDrive│
                     └────────┬────────┘
                              │
                     Left / Right Wheels
                              │
                              ▼
                           Robot
                              │
              ┌───────────────┼────────────────┐
              │               │                │
            /odom           /tf              /scan
              │               │                │
              └───────────────┼────────────────┘
                              ▼
                            ROS 2
                              │
                              ▼
                            RViz
                              │
                        Future SLAM
                              │
                       Future Nav2
```

---

# 26. Key Commands — Quick Reference

### Source ROS 2

```bash
source /opt/ros/jazzy/setup.bash
```

### Source workspace

```bash
source ~/ws_mobile/install/setup.bash
```

### Build

```bash
cd ~/ws_mobile
colcon build
```

### Launch

```bash
ros2 launch mobile_robot gazebo_model.launch.py
```

### Teleoperate

```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

### Run RViz

```bash
rviz2
```

### List ROS 2 topics

```bash
ros2 topic list
```

### Check LiDAR

```bash
ros2 topic list | grep scan
ros2 topic type /scan
ros2 topic echo /scan --once
```

### List Gazebo topics

```bash
gz topic -l
```

### Search Gazebo LiDAR topics

```bash
gz topic -l | grep -Ei "scan|lidar"
```

---

# 27. Viva-Level Concepts

### Degree of Freedom

Planar robot motion has:

```text
X, Y, Yaw
```

Therefore:

```text
3 DOF
```

The differential-drive mechanism uses two independently actuated wheel rotations.

### LiDAR

Measures distance to surrounding geometry using multiple laser rays.

### `/scan`

ROS 2 LiDAR topic carrying:

```text
sensor_msgs/msg/LaserScan
```

### TF

Maintains relationships between coordinate frames.

### `/cmd_vel`

Carries linear and angular velocity commands using:

```text
geometry_msgs/msg/Twist
```

### Gazebo

Provides physics, environment, collisions and simulated sensors.

### RViz

Visualizes ROS 2 robot state, sensor data, TF, maps and other interfaces.

### `ros_gz_bridge`

Connects Gazebo Transport messages with ROS 2 messages.

### Xacro

Makes robot descriptions parameterized and easier to maintain than a fully expanded URDF.

### `robot_state_publisher`

Publishes TF transforms derived from the robot description and joint states.

### SLAM

Simultaneously estimates robot pose and builds an environment map.

---

# 28. Current Project Status

### Implemented

- [x] ROS 2 Jazzy environment
- [x] Gazebo Harmonic environment
- [x] Differential-drive robot model
- [x] Xacro/URDF description
- [x] Wheel joints and drive configuration
- [x] Caster
- [x] Gazebo DiffDrive plugin
- [x] Robot state publishing
- [x] Joint-state publishing
- [x] Odometry
- [x] TF
- [x] ROS 2 ↔ Gazebo bridge
- [x] Keyboard teleoperation
- [x] Simulated GPU LiDAR
- [x] `/scan` LaserScan bridge

### Next milestones

- [ ] RViz LiDAR visualization
- [ ] IMU integration
- [ ] SLAM
- [ ] Map generation
- [ ] Nav2
- [ ] Autonomous navigation
- [ ] Physical robot implementation

---

## Author

**Arshit Gautam**  
B.Tech — Computer Science & Engineering (Artificial Intelligence & Machine Learning)

Developed as a ROS 2 robotics project with a focus on mobile robotics, simulation, perception, control, and future hardware integration.

---

## Technology Stack

```text
ROS 2 Jazzy
Gazebo Harmonic
Xacro / URDF
ros_gz_bridge
CMake / ament
Python
RViz
LiDAR
Differential Drive
SLAM (planned)
Nav2 (planned)
```
