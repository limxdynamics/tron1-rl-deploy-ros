<p align="center">
  <img src="doc/banner.png" alt="TRON1 RL Deployment: ONNX walking policies for pointfoot, solefoot and wheelfoot legs" width="100%">
</p>

<p align="center">
  <b>English</b> | <a href="README_cn.md">中文</a>
</p>

# TRON1 RL Deployment — ROS Noetic

> 🧭 **Part of the TRON1 open-source ecosystem.** All TRON1 repositories and where to start: [TRON1 open-source ecosystem](https://github.com/limxdynamics/tron1-rl-isaacgym#tron1-open-source-ecosystem).

Deploy trained reinforcement learning (RL) walking policies to TRON1 biped and wheel-legged robots via ROS Noetic. Loads ONNX models through `ros_control` and runs them on Gazebo simulation or real hardware.

## How It Works

<p align="center">
  <img src="doc/figures/pipeline.png" alt="policy.onnx and encoder.onnx feed controller_manager, which drives either Gazebo or the real TRON1 through the same plugin" width="100%">
</p>

<p align="center"><sub>Two launch files, one policy. Only the hardware interface changes.</sub></p>


The launch files automatically load the ONNX policy from `robot_controllers/config/pointfoot/<ROBOT_TYPE>/policy/<RL_TYPE>/` based on your `ROBOT_TYPE` and `RL_TYPE` environment variables.

## 1. Prerequisites

- Ubuntu 20.04
- ROS Noetic (`ros-noetic-desktop-full`)
- ONNX Runtime 1.10.0

### Install ROS dependencies

```bash
sudo apt-get update
sudo apt install ros-noetic-urdf ros-noetic-kdl-parser ros-noetic-hardware-interface \
    ros-noetic-controller-manager ros-noetic-controller-interface ros-noetic-ros-control \
    ros-noetic-gazebo-* ros-noetic-robot-state-* ros-noetic-joint-state-* \
    ros-noetic-rqt-gui ros-noetic-rqt-controller-manager ros-noetic-plotjuggler* \
    cmake build-essential libpcl-dev libeigen3-dev libopencv-dev libmatio-dev \
    python3-pip libboost-all-dev libtbb-dev liburdfdom-dev liborocos-kdl-dev -y
```

### Install ONNX Runtime

```bash
wget https://github.com/microsoft/onnxruntime/releases/download/v1.10.0/onnxruntime-linux-x64-1.10.0.tgz
tar xvf onnxruntime-linux-x64-1.10.0.tgz
sudo cp -a onnxruntime-linux-x64-1.10.0/include/* /usr/include
sudo cp -a onnxruntime-linux-x64-1.10.0/lib/* /usr/lib
```

## 2. Build

```bash
mkdir -p ~/limx_ws/src && cd ~/limx_ws/src
git clone https://github.com/limxdynamics/limxsdk-lowlevel.git
git clone https://github.com/limxdynamics/tron1-gazebo-ros.git
git clone https://github.com/limxdynamics/robot-description.git
git clone https://github.com/limxdynamics/robot-visualization.git
git clone https://github.com/limxdynamics/tron1-rl-deploy-ros.git

cd ~/limx_ws
catkin_make install
source install/setup.bash
```

## 3. Run Simulation

### Set robot type and training environment

Available robot types: `PF_P441A`, `PF_P441B`, `PF_P441C`, `PF_P441C2`, `PF_TRON1A`, `SF_TRON1A`, `WF_TRON1A`

Supported training environments: `isaacgym`, `isaaclab`

```bash
echo 'export ROBOT_TYPE=PF_P441A' >> ~/.bashrc
echo 'export RL_TYPE=isaacgym' >> ~/.bashrc
source ~/.bashrc
```

Pre-trained ONNX policies are included for most but not all combinations. If your combination lacks a policy, see [Deploy Your Own Policy](#5-deploy-your-own-policy).

<p align="center">
  <img src="doc/figures/policy-layout.png" alt="ROBOT_TYPE and RL_TYPE expand into the policy file path, with a worked example for PF_TRON1A and isaacgym" width="100%">
</p>

<p align="center"><sub><code>ROBOT_TYPE</code> and <code>RL_TYPE</code> expand into the file path; the same layout is where your own ONNX pair goes.</sub></p>


### Launch

```bash
source ~/limx_ws/install/setup.bash
roslaunch robot_hw pointfoot_hw_sim.launch
```

This starts Gazebo with the robot, loads the ONNX RL policy, and begins inference. The robot should begin walking automatically.

### Launch Flags

| Flag | Default | Description |
|------|---------|-------------|
| `plot` | `false` | Set to `true` to enable PlotJuggler data visualization |

## 4. Deploy to Real Robot

```bash
source ~/limx_ws/install/setup.bash
roslaunch robot_hw pointfoot_hw.launch
```

This loads the same ONNX policy but communicates with real hardware instead of Gazebo.

Set `rviz:=true` to enable RViz visualization:

```bash
roslaunch robot_hw pointfoot_hw.launch rviz:=true
```

## 5. Deploy Your Own Policy

Place your trained ONNX files at the expected path:

```
robot_controllers/config/pointfoot/<ROBOT_TYPE>/policy/<RL_TYPE>/
    policy.onnx
    encoder.onnx
```

Then launch as usual — your custom policy will be loaded automatically.

<p align="center">
  <img src="doc/figures/control-loop.png" alt="One 20 ms policy step made of ten 2 ms hardware cycles, and the observation to joint-target chain inside it" width="100%">
</p>

<p align="center"><sub>What the controller does every 20 ms: observations in, joint targets out.</sub></p>


## 6. Related Repositories

- [tron1-mujoco-sim](https://github.com/limxdynamics/tron1-mujoco-sim) — MuJoCo simulation for policy validation
- [tron1-rl-deploy-python](https://github.com/limxdynamics/tron1-rl-deploy-python) — Python-based RL deployment
- [tron1-rl-deploy-ros2](https://github.com/limxdynamics/tron1-rl-deploy-ros2) — ROS 2 Iron version
- [tron1-robot-description](https://github.com/limxdynamics/tron1-robot-description) — URDF/Xacro robot models
