# The rb1_ros2_description package

This package contains a ROS2 robot model description for Robotnik's RB-1 mobile base. It spawns the robot in Gazebo and sets up two controllers:

- `rb1_base_controller` (differential drive): drives the robot.
- `elevator_controller`: moves the robot's lifting unit up and down.

## Disclaimer:
This package only modifies/adapts files from these repositories/packages:
- [RobotnikAutomation/rb1_base_sim](https://github.com/RobotnikAutomation/rb1_base_sim) licensed under the BSD 2-Clause "Simplified" License
- [RobotnikAutomation/rb1_base_common/rb1_base_description](https://github.com/RobotnikAutomation/rb1_base_common/tree/melodic-devel/rb1_base_description), licensed under the BSD License
- [RobotnikAutomation/robotnik_sensors](https://github.com/RobotnikAutomation/robotnik_sensors) licensed under the BSD License

## Installation

Prerequisites: Ubuntu 24.04 and ROS 2 Jazzy.

```bash
sudo apt update
sudo apt install ros-jazzy-gz-ros2-control ros-jazzy-ros2-controllers ros-jazzy-ros2-control ros-jazzy-teleop-twist-keyboard
```

Clone this package into a workspace, then build:

```bash
cd ~/ros2_ws/src
git clone https://github.com/jarrebear/rb1_control_checkpoint.git
cd ~/ros2_ws
colcon build
source install/setup.bash
```

## Getting started

Launch the simulation:

```bash
ros2 launch rb1_ros2_description rb1_ros2_xacro.launch.py
```

Gazebo opens with the robot in an empty world.

## Controller activation

The launch file activates the controllers automatically. In a second terminal, check that they are active:

```bash
ros2 control list_controllers
```

`rb1_base_controller` and `elevator_controller` should both show `active`.

If one is missing, activate it manually:

```bash
ros2 run controller_manager spawner rb1_base_controller --controller-manager /controller_manager
ros2 run controller_manager spawner elevator_controller --controller-manager /controller_manager
```

## Moving the robot around

In a new terminal, run the keyboard teleop, remapped to the differential drive controller's command topic:

```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard --ros-args \
  -p stamped:=true \
  -r cmd_vel:=/rb1_base_controller/cmd_vel
```

## Moving the robot's lifting unit

The lifting unit is controlled with a force command (effort) on the `/elevator_controller/commands` topic. A positive value moves it up. A value of `0.0` applies no force, so the platform lowers under its own weight.

Raise:

```bash
ros2 topic pub --once /elevator_controller/commands std_msgs/msg/Float64MultiArray "{data: [0.05]}"
```

Lower

```bash
ros2 topic pub --once /elevator_controller/commands std_msgs/msg/Float64MultiArray "{data: [0.0]}"
```

The platform moves until it reaches its upper or lower limit. To watch its position, run:

```bash
ros2 topic echo /joint_states --field position
```

The first value is the lifting unit, in metres.