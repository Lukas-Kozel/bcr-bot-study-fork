# Developer Notes & Cheat Sheet: BCR Bot Study Fork

This repository serves as an educational sandbox for developing and testing real-time ROS 2 algorithms (Kalman Filters, PID control, Sensor Fusion, and Computer Vision) using the `bcr_bot` platform.

## 📁 Repository Structure Overview

Understanding the directory structure is crucial for modifying the robot or adding new features.

| Directory / File | Description |
| :--- | :--- |
| **`config/`** | Contains YAML configuration files (e.g., RViz settings, `ros_gz_bridge` mappings, Nav2 parameters). |
| **`launch/`** | Python launch files (`.launch.py`) used to bring up the simulation, spawn the robot, and start the bridges. |
| **`meshes/`** | Raw 3D geometry files (e.g., `.dae`, `.stl`) for the visual and collision properties of the robot parts. |
| **`models/`** | SDF models for Gazebo Garden (objects, obstacles, etc.). |
| **`worlds/`** | Gazebo world files (e.g., `small_warehouse.sdf`) defining the simulation environment. |
| **`urdf/`** | XACRO and URDF files defining the robot's kinematics, joints, links, and sensor plugins. |
| **`rviz/`** | Pre-configured `.rviz` layouts for visualizing sensor data (LiDAR, Cameras, TF). |
| **`scripts/`** | Executable Python scripts and utility nodes. |
| **`CMakeLists.txt`** | Build configuration for `colcon`. Defines how the package is compiled and what files are installed. |
| **`package.xml`** | Defines package dependencies (`rosdep` uses this to install required system libraries). |

---

## 🛠️ Build Instructions

Always build the workspace from the root of the workspace (e.g., `~/robotics_ws`), **not** from inside the package folder.

### 1. Install Dependencies
Make sure you have installed the Gazebo Garden bridge and all ROS dependencies:

```bash
sudo apt-get update
sudo apt-get install -y ros-humble-ros-gzgarden ros-humble-ros-gzgarden-bridge ros-humble-ros-gzgarden-sim ros-humble-ros-gzgarden-interfaces
cd ~/robotics_ws
rosdep install --from-paths src --ignore-src -r -y
```

### 2. Build the Package
We use `--symlink-install` so Python scripts and URDF changes reflect immediately without needing a rebuild.

```bash
cd ~/robotics_ws
source /opt/ros/humble/setup.bash
export GZ_VERSION=garden

colcon build --symlink-install --packages-select bcr_bot --cmake-args -DCMAKE_BUILD_TYPE=Release
```

---

## 🚀 Running the Simulation

You will need multiple terminal tabs to run the simulation and control the robot.

### Terminal 1: Launch Gazebo & The Robot
This launches the environment, spawns the robot, and starts the `ros_gz_bridge`.

```bash
cd ~/robotics_ws
source /opt/ros/humble/setup.bash
source install/setup.bash
export GZ_VERSION=garden

ros2 launch bcr_bot gz.launch.py camera_enabled:=True two_d_lidar_enabled:=True world_file:=small_warehouse.sdf
```

### Terminal 2: Teleop (Keyboard Control)
Use this node to drive the robot around the warehouse to test sensor data gathering.

```bash
source /opt/ros/humble/setup.bash

# We remap the default cmd_vel to the robot's specific namespace
ros2 run teleop_twist_keyboard teleop_twist_keyboard cmd_vel:=/bcr_bot/cmd_vel
```

### Terminal 3: RViz2 (Visualization)
To visualize LiDAR scans, camera feeds, and the TF tree.

```bash
source /opt/ros/humble/setup.bash
source ~/robotics_ws/install/setup.bash

rviz2 -d ~/robotics_ws/src/bcr_bot/rviz/bcr_bot.rviz
```

---

## 📡 Sensor & Topic Cheat Sheet

Once the simulation is running, you can subscribe to the following topics to feed data into your algorithms (e.g., Kalman Filter).

| Sensor Type | ROS 2 Topic | Message Type | Notes |
| :--- | :--- | :--- | :--- |
| **Wheel Odometry** | `/bcr_bot/odom` | `nav_msgs/msg/Odometry` | Kinematic movement estimation. |
| **IMU** | `/bcr_bot/imu` | `sensor_msgs/msg/Imu` | Accelerations & Angular velocities. |
| **GPS (Custom)** | `/bcr_bot/gps/fix` | `sensor_msgs/msg/NavSatFix` | Absolute global positioning. |
| **2D LiDAR** | `/bcr_bot/scan` | `sensor_msgs/msg/LaserScan` | Obstacle detection, 2D SLAM. |
| **RGB Camera** | `/bcr_bot/camera/image_raw`| `sensor_msgs/msg/Image` | Computer Vision tasks. |
| **Control Input** | `/bcr_bot/cmd_vel` | `geometry_msgs/msg/Twist` | Publish here to move the robot. |

*To inspect live data from any sensor in the terminal, use:*
```bash
ros2 topic echo <topic_name>
# Example: ros2 topic echo /bcr_bot/imu
```

---

## 🧠 Algorithm Development Workflow

When creating new algorithms (like an EKF or PID Controller):
1. **Do not** modify the `bcr_bot` source code directly unless adding hardware (like GPS).
2. Create a **new ROS 2 package** in `~/robotics_ws/src` (e.g., `ros2 pkg create --build-type ament_python algo_sandbox`).
3. Write nodes that **Subscribe** to the sensors listed above, process the data, and **Publish** velocity commands back to `/bcr_bot/cmd_vel`.
