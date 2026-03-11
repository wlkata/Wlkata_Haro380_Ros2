**English** | [中文 / Chinese](README-ZH.md)

## Project Overview

This project is an integrated control and teaching system for the **WLKATA Haro380 series robotic arm**, mainly including:

- **Robot model and description** (`wlkata_harobot_description`): URDF/Xacro, joint configuration, etc., used to display the robot in RViz / simulation.
- **MoveIt motion planning configuration** (`wlkata_harobot_moveit_config`): MoveIt 2 configuration package for path planning and visualization of the Haro380.
- **Real robot control node** (`wlkata_arm_move`): Subscribes to trajectories planned by MoveIt, converts them to G-code, and sends them to the physical robot via serial port.
- **Python control SDK** (`wlkatapython`): A standalone Python package that controls Harobot, E4, MT4, MS4220 and other devices via serial + G-code.

Typical use cases:

- Use ROS 2 + MoveIt for **Haro380 motion planning and visualization**.
- **Seamlessly send MoveIt planned trajectories to the real robot**.
- Perform **secondary development / upper-computer programming** based on `wlkatapython`.


## Directory Structure

Key project directories:

- `wlkata_harobot_description/`
  - `urdf/`: `wlkata_harobot_description.urdf`, `.xacro` and other robot model files.
  - `config/`: YAML files for joint names and configurations.
  - `launch/display.launch.py`: Display the Haro380 model in RViz.

- `wlkata_harobot_moveit_config/`
  - `config/`: MoveIt configuration files such as `kinematics.yaml`, `ros2_controllers.yaml`, `wlkata_harobot_description.srdf`, etc.
  - `launch/`:
    - `demo.launch.py`: Common MoveIt + RViz demo entry.
    - `move_group.launch.py`, `moveit_rviz.launch.py`, `rsp.launch.py`, etc.
  - `package.xml`: MoveIt configuration package definition.

- `wlkata_arm_move/`
  - `wlkata_arm_move/harobot_moveit_move.py`: Core Python node that subscribes to `/display_planned_path` (`moveit_msgs/DisplayTrajectory`), converts the planned joint trajectory into G-code commands, and sends them to the robot via serial port.
  - `package.xml`: ROS 2 Python package definition, depending on `rclpy`, `moveit_msgs`, `wlkata_interfaces`, `action_msgs`, `std_msgs`, etc.
  - `setup.py`: Exposes the executable node `harobot_moveit_move` via `entry_points`.

- `wlkatapython/`
  - `wlkatapython.py` and multiple subdirectories (`Harobot_robot/`, `E4_robot/`, `MT4_robot/`, `MS4220_robot/`, etc.): Serial control implementation for various devices.
  - `README.md`: Detailed introduction on how to control Haro380/E4/MT4/MS4220 and other devices using Python + serial + G-code (including sample code and wiring diagrams).
  - `setup.py`: Packaging configuration of the standalone Python package, which can be used by installing with `pip`.


## Environment Requirements

- **Operating system**: Ubuntu 22.04 / 24.04 that supports ROS 2 + MoveIt 2 is recommended.
- **ROS 2 + MoveIt 2**: Includes dependencies such as `ament_cmake`, `moveit_ros_move_group`, `moveit_ros_visualization`, `robot_state_publisher`, `rviz2`, etc. (see each `package.xml`).
- **Python environment**: Python ≥ 3.9.
- **Build tools**: `colcon`.


## Workspace Layout and Where to Run Commands

Recommended ROS 2 workspace layout (example):

```text
~/ros2_ws/
  ├─ src/
  │   ├─ wlkata_harobot_description/
  │   ├─ wlkata_harobot_moveit_config/
  │   ├─ wlkata_arm_move/
  │   └─ wlkatapython/
  └─ ...
```

- **Where to place this repository**: Put the whole project folder under `~/ros2_ws/src/` (so that you can see the above packages directly under `src/`).
- **Where to run commands**:
  - `sudo apt install ...`: Can be run from any directory.
  - `cd ~/ros2_ws`: Go to the ROS 2 workspace root.
  - `colcon build`: Run in `~/ros2_ws`.
  - `source install/setup.bash`: Run in `~/ros2_ws` or any other terminal to load the workspace environment.
  - `ros2 launch ...` / `ros2 run ...`: Run in a terminal where `source install/setup.bash` has already been executed (current directory does not matter).


## Quick Start (Minimal)

### Ubuntu 24.04 + ROS 2 Jazzy

```bash
sudo apt install \
  ros-jazzy-joint-state-publisher-gui \
  ros-jazzy-moveit \
  ros-jazzy-controller-manager \
  ros-jazzy-joint-trajectory-controller \
  ros-jazzy-joint-state-broadcaster

cd ~/ros2_ws          # replace with your workspace
colcon build
source install/setup.bash

ros2 launch wlkata_harobot_description display.launch.py
ros2 launch wlkata_harobot_moveit_config demo.launch.py
```

- `ros2 launch wlkata_harobot_description display.launch.py`: Only launches the robot description and RViz. You can see the **Haro380 model**, joint state publisher/GUI, etc. Mainly used to check whether URDF and joint configuration are correct. It does not include the MoveIt planning interface.
- `ros2 launch wlkata_harobot_moveit_config demo.launch.py`: Launches the **MoveIt + RViz demo scene**. The MotionPlanning panel will appear on the left side in RViz, and you can interactively plan and execute trajectories (in simulation or together with the real robot node).


### Ubuntu 22.04 + ROS 2 Humble

Package names are basically the same, just replace the `ros-jazzy-*` prefix with `ros-humble-*`:

```bash
sudo apt install \
  ros-humble-joint-state-publisher-gui \
  ros-humble-moveit \
  ros-humble-controller-manager \
  ros-humble-joint-trajectory-controller \
  ros-humble-joint-state-broadcaster

cd ~/ros2_ws          # replace with your workspace
colcon build
source install/setup.bash

ros2 launch wlkata_harobot_description display.launch.py
ros2 launch wlkata_harobot_moveit_config demo.launch.py
```


## Dependency Notes on Ubuntu 22.04 / 24.04 (Brief)

- Ubuntu 24.04 + ROS 2 Jazzy: Core dependencies can be installed via the `apt` command above. For development, you may additionally install `python3-colcon-common-extensions`, `python3-rosdep`, etc.
- Ubuntu 22.04 + ROS 2 Humble: Replace all `ros-jazzy-*` package names with `ros-humble-*`. If ROS 2 is not yet installed, it is recommended to first install the desktop version following the official ROS documentation.


## Serial Port and Controller Notes

The serial communication part mainly refers to `wlkatapython/README.md`:

- Use **G-code protocol + serial communication (RS485/UART)** to control the robot arm, linear slide, conveyor belt and other devices.
- Usually you need a **multi-function controller** to fully use all features. If the robot is connected directly via serial port, some functions may not be available.

Example (UART mode, Haro380 homing):

```python
import wlkatapython
import serial

Serial1 = serial.Serial("/dev/ttyUSB0", 115200)
harobot1 = wlkatapython.Harobot_UART()
harobot1.init(Serial1, -1)
harobot1.homing()
Serial1.close()
```

In `harobot_moveit_move.py`, the default serial device is:

```python
serial.Serial("/dev/ttyUSB0", 115200, timeout=1)
```

- Please confirm the actual serial port name (typically `/dev/ttyUSB0`, `/dev/ttyUSB1`, etc.) and modify according to your environment.


## Brief Description of Key Packages

### `wlkata_harobot_description`

- Provides the Harobot robot model description (URDF/Xacro), joint configuration, and Launch files for display/simulation. Can be used to verify whether the model and coordinate frames are correct.

### `wlkata_harobot_moveit_config`

- Generated using MoveIt Setup Assistant. Contains kinematics, controllers, planning scene and other configurations. Provides entries such as `demo.launch.py` to start the MoveIt + RViz demo scene.

### `wlkata_arm_move`

- ROS 2 Python package. The core node `harobot_moveit_move` receives trajectories planned by MoveIt, converts them into G-code that conforms to the WLKATA controller, sends them to the physical robot via serial port, and handles responses.

### `wlkatapython`

- Standalone Python package that can be used by installing with `pip`. Supports multiple devices including Harobot, E4, MT4, MS4220, and supports both RS485 and UART communication. Its `README.md` contains many examples and wiring diagrams. The ROS 2 control nodes in this project directly reuse its communication logic.