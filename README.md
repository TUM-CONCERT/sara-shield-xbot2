# SaRA-shield-xbot2
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

The [SaRa-shield](https://github.com/JakobThumm/sara-shield) provides safety for human-robot interaction using reachability analysis.
We use [SaRA](https://github.com/Sven-Schepp/SaRA) to calculate the reachable sets of humans and robots.
The SaRA shield additionally provides the necessary trajectory control to stop the robot before any collision with the human could occur.


This is a fork of https://github.com/JakobThumm/sara-shield, which combines sara-shield with the real-time framework [xbot2](https://advrhumanoids.github.io/xbot2/). The main code for this integration is in `safety_shield/src/sara_shield_xbot2.cc` and `safety_shield/include/safety_shield/sara_shield_xbot2.h` and works as follows: 
1. Sara-shield gets created
2. We subscribe to rostopics. Over ROS, this plugin receives:
    - Pose of humans
    - Goal Joint Positions of the robot OR a trajectory
3. While the rosnode is running, the following happens in each timestep:
    1. If a new goal state is received, make a new `newLongTermTrajectory`
    2. Perform one step of the sara-shield, which returns a joint positions
    3. Use the aquired joint positions to control the robot over xbot2


# Installation

The recommended way to set up the CONCERT-sara-shield project is listen [here](https://github.com/TUM-CONCERT/sara-shield-stack-CONCERT). If you only want to set up this project, follow the instructions below. 

### Clone the repo with submodules
```
git clone --recurse-submodules https://github.com/TUM-CONCERT/sara-shield-xbot2.git
```
### Install the shield [C++ only]
The installation requires `gcc`, `c++>=17`, and `Eigen3` version 3.4 (download it here: https://eigen.tuxfamily.org/index.php?title=Main_Page).
Set the path to your eigen3 installation to this env variable, e.g.,
```
export EIGEN3_INCLUDE_DIR="/usr/include/eigen3/eigen-3.4.0"
```
Install gtest
```
sudo apt-get install libgtest-dev
```
```
cd safety_shield
mkdir build && cd build
cmake ..
make -j 4
```
### Install the shield [With Python bindings]
```
pip install -r requirements.txt
python setup.py install
```
### Run the python binding tests
```
pytest safety_shield/tests
```

# Run Safety-shield with Xbot2
Start Ros with
```
roscore
```
Run the concert gazebo project with
```
cd ~/concert_ws
source setup.bash
mon launch sara_shield concert.launch
```
**After** Gazebo runs, start Xbot2-Gui with
```
xbot2-gui
```
This should open a large window with status "Running" in the top left corner. If it opens a small window with the status "Inactive" instead, close and rerun the command above.

**Open Rviz** for visualization (with the command ```rviz```) and **start the xbot plugin** by pressing the button `sara_shield_xbot2` in the xbot2 gui. The visualization topics are listed in the namespace `sara_shield\*` and can be visualized by adding them in Rviz.

## Dependencies:
- CONCERT: [concert_msgs](https://github.com/ADVRHumanoids/concert_msgs) : Defines a ros message for Humans, using keypoints 
- Other: 
    
   `gazebo_ros`: If the plugin is run in simulation, `gazebo_msgs` are parsed to get the relative transformation between the robot (`base_link`) and the `world` frame   
    `xbot2` : Used to communicate with the robot    
    `Eigen3.4`: Used for internal calculations


## Building after Modifying
If the project is build according to https://github.com/TUM-CONCERT/sara-shield-stack-CONCERT, (re-)building should be done in the following way:
```
cd /home/user/tum_integration_ws/build/sara-shield
cmake ../../src/sara-shield/safety_shield/
make -j4 install
```

## Notes
1. Sara-shield should always be built in *Release* mode, since the timesteps can take too long otherwise, resulting in crashes (```safety limit violation detected ...```).
2. The capsules have to be computed saperately for every robot configuration
