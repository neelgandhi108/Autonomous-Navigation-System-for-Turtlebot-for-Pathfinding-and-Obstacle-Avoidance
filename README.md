
# Autonomous Navigation System for Turtlebot for Pathfinding and Obstacle Avoidance

<!-- <div align="center"> <img src="imagelink"><br><br> </div> -->

*Note: Be sure to have ROS Kinetic Kame installed with the `desktop-full` option. You will also need the `ros-kinetic-turtlebot` package.

Please consider further refining my Pathfinding and Obstacle Avoidance Algorithm.

## Project Synopsis

The project aim is to develop an autonomous navigation mechanism for the Turtlebot in ROS Gazebo, capable of identifying the optimal route and dodging obstacles within a labyrinthine setting. For this, we've applied the A* search algorithm, which directs the robot's path to the target based on the data captured by its RGB-D Kinect sensor. This system can tackle both familiar and unfamiliar terrains as long as the initial and final coordinates are (0,0) and (4,4), respectively.

To incorporate this repository in a new or existing catkin workspace, refer to the instructions given below.

## Preparation & Configuration

### Integrating this repository with a **new** catkin workspace

1.  Establish a new catkin workspace.
    
    Open your terminal shell, move to your selected directory for the new workspace, and execute the following commands. Replace `<wsname>` with your chosen workspace name.
    
    shellCopy code
    
    `$ mkdir -p <wsname>/src
    $ cd <wsname>/src/
    $ catkin_init_workspace
    $ cd ..
    $ catkin_make` 
    
2.  Clone this repository and transfer necessary folders into your catkin workspace.
    
    In the same `<wsname>` directory, execute the following:
    
    shellCopy code
    
    `$ git clone [https://github.com/neelgandhi108/Autonomous-Navigation-System-for-Turtlebot-for-Pathfinding-and-Obstacle-Avoidance.git](https://github.com/neelgandhi108/Autonomous-Navigation-System-for-Turtlebot-for-Pathfinding-and-Obstacle-Avoidance/)
    $ cd Autonomous-Navigation-System-for-Turtlebot-for-Pathfinding-and-Obstacle-Avoidance/
    $ cp -r src/bot ../src
    $ cp -r worlds docs project_init.sh ../
    $ cd ..
    $ catkin_make` 
    

### Integrating this repository with an **existing** catkin workspace

1.  Clone this repository and transfer necessary folders into your existing catkin workspace.
    
    In your terminal shell, navigate to the root directory of your existing catkin workspace and execute the following:
    
    shellCopy code
    
    `$ git clone [https://github.com/neelgandhi108/Autonomous-Navigation-System-for-Turtlebot-for-Pathfinding-and-Obstacle-Avoidance.git](https://github.com/neelgandhi108/Autonomous-Navigation-System-for-Turtlebot-for-Pathfinding-and-Obstacle-Avoidance/)
    $ cd Autonomous-Navigation-System-for-Turtlebot-for-Pathfinding-and-Obstacle-Avoidance/
    $ cp -r src/bot ../src
    $ cp -r worlds docs project_init.sh ../
    $ cd ..
    $ catkin_make` 
    

## Running the Code

### Initializing the Gazebo environment:

To set up the Gazebo world, execute the following commands in a terminal:

shellCopy code

`$ source devel/setup.bash
$ chmod +x project_init.sh
$ ./project_init.sh` 

### Starting autonomous navigation to goal:

Open a new terminal, execute the following commands to start the Turtlebot's autonomous navigation.

shellCopy code

`$ source devel/setup.bash
$ roslaunch bot bot.launch` 

Upon reaching the goal coordinates, the Turtlebot will display a notification in the terminal along with the time it took to reach the target.

----------

## Code Analysis & Clarifications

### Nodes

We developed four main nodes for this package that handle different aspects of the Turtlebot's autonomous navigation.

1.  `pos_info`
    
    This node subscribes to the `odom_combined` topic published by the `robot_pose_ekf` node. It's a refined version of standard odometry provided by `\odom` as it merges Extended Kalman Filtering of the odometry data with Inertial Measurement Unit (IMU) data from the Turtlebot. It then publishes the Turtlebot's pose estimate for other nodes to subscribe to.
    
2.  `depth_info`
    
    This node publishes the depth information from the Turtlebot's RGB-D Kinect sensor at the center point. This acts as our basic obstacle detection system for barriers directly in the Turtlebot's path.
    
3.  `scan_info`
    
    This node subscribes to the `\scan` topic published by the `laserscan_nodelet_manager` node that obtains the scan depth information from the `depthimage_to_laserscan`. This transforms the RGB-D Kinect sensor data into 'virtual' laser readings akin to that of a LIDAR sensor. This node then publishes the depth information for the left-most, right-most, and middle readings on the horizontal axis. The `bot_control` node uses this data to act as our anticipatory obstacle detection system for obstructions along the Turtlebot's intended path.
    
4.  `bot_control`
    
    This node serves as our primary node that manages the motion control, obstacle evasion, and pathfinding for the Turtlebot. It subscribes to the `/auto_ctrl/scan_info`, `/auto_ctrl/depth_info`, and `/auto_ctrl/pos_info` topics. Using the algorithms found in the `algo.h` header file, it determines the next coordinate to move towards, dodging obstacles, updating the algorithm with the presence of obstacles to reach the goal coordinates.
    

### Algorithm file `algo.h`

This file contains two algorithms, Flood Fill and A* search. However, we primarily utilize the A* search algorithm due to its speedy computation and reliability. This file also holds the code for the Turtlebot's internal 2D array map so that the algorithm can compute the next move.

### Launch files

1.  `world.launch`
    
    This launch file gets started in the `project_init.sh` shell file to launch the Gazebo environment using the defined worlds in `project_init.sh` as well as the Turtlebot base. It also includes the `empty_world.launch` file which sets the different arguments for the Gazebo launch.
    
2.  `empty_world.launch`
    
    We added the line `<remap from="tf" to="gazebo_tf"/>` in this launch file to ensure that the tf tree created by Gazebo does not interfere with the tf tree created by Turtlebot. We needed to do this to use the `robot_pose_ekf` node as there was a conflict in the tf frames created by Gazebo and the `robot_pose_ekf` node which prevented `\odom_combined` from connecting to the tf tree.
    
3.  `bot.launch`
    
    This launch file is our primary launch file to run the `robot_pose_ekf` node and the four nodes that we mentioned previously.
    

### Choosing the worlds for testing

To select the world for testing the Turtlebot performance, you need to amend the `project_init.sh` file to change the chosen world file.

1.  `test_world_1.world`
    
    Time taken to reach goal: ~50s
    
    This world is the simplest one to test our system as it has the shortest and easiest path in terms of obstacle evasion to guide the Turtlebot from start to goal.
    
       ![World 1](/worlds/World1_sample_images/test-wolrd-1-top-view.jpg)
    
2.  `test_world_2.world`
    
    Time taken to reach goal: ~100s
    
    This world has additional complexity due to more obstacles to be avoided and a longer path to travel from start to goal, which increases the chance of the Turtlebot drifting off its intended course.
    
    ![World 2](/worlds/World2_sample_images/world_2_snapshot.jpg)
    
3.  `test_world_trap.world`
    
    Time taken to reach goal: ~2000s
    
    We designed this world to challenge our system by adding five potential dead ends along the Turtlebot's path and to test if it can recover after being trapped in the dead end and navigate to the goal.
    
      ![World 3](/worlds/World3_sample_images/world_3_top.jpg)
    

## Acknowledgments

This project was guided by tutorials from the course EE4308 - Advances in Intelligent Systems and Robotics at the National University of Singapore.

**System Prerequisites**

**Version / Release**

Operating System

[Ubuntu 16.04 LTS (Xenial Xerus)](http://releases.ubuntu.com/16.04/)

ROS Distribution Release

[Kinetic Kame](http://wiki.ros.org/kinetic/Installation/Ubuntu)*

C++ Version

Minimum C++11

All dependencies have been updated and resolved as of June 22, 2023.
