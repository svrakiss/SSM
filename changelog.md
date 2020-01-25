#1/12/20
+ 7 hours trying to get demos to work.
#1/13/20
1 hour still trying to get ros with realsense to work.
Intel: They actually moved it to a side project. 
 
https://support.intelrealsense.com/hc/en-us/community/posts/360033595714-D435-USB-connection-issues
Contains info about connecting realsense cameras to ethernet and the problems when using cheap usb cables

#1/14/20
3 hours working on fixing realsense bugs (usb driver issues in ros, tried to get a torch2trt example 
#1/15/20 
7 hours 35 min 
+ 2 hours 22 min
#1/16/20
+44 min 
#1/17/20
1 hour 35 min 
Figuring out how to transfer openpose params to trt_pose example. This is so that we don't have to rely on installing openpose or ros.
Note: Jetson Xavier clocked 1k+ fps running a Pytorch model optimized using torch2trt. 
+3 hour 50 min 
Openpose automatically pads and resizes the input image to match the resolution of the network. AND it scales the heatmaps back to the original image.
That is necessary for this situation.
Going to use lightnet to **letterbox** input (resize & pad image while preserving aspect ratio)
#1/18/20
+ 1 hour 21 min
Timer that shuts down pipeline and closes opencv display works now.
Confirmed that images are being letterboxed. Next step is to figure out how to rescale the heatmaps and part affinity fields.
+3 hours and 9 minutes
Isaac works and I figured out how to get it to scale the images, preserve the aspect ratio, and rescale the keypoints to the original resolution


#1/19/20
+ 1 hour 20 min
Task at hand: Take in Skeleton2ListProto and aligned depth image, output Skeleton3ListProto.
I need to create a codelet that does this. Not immediately clear what I should be doing. Add at least 4 hours to the development time because bazel needs to compile a new library.
+ 1hour 13 min
Unable to view ISAAC source code, so this will be more difficult. Saw another solution from LIPS that incorporates skeleton tracking and localization with the realsense. Support for the ISAAC is on their github page github.com/lips-hci/stereo_ae400 for their industrial grade realsense camera. Note their camera case has IP67 rated protection.

+ 50 min
Found support for Openni2 on jetson nano and ubuntu 18 (on structure.io)
Note: Librealsense cannot be installed on VMware Workstation 15 Player. Libglfw3 cannot be installed. Next step would be to dualboot the computer.
Going to make a github repo for code to make it portable.
+ 1 hour 37 min
Brainstorming ideas for velocity estimation.

+ 1 hour adding openni2 support for realsense

#1/21/20
+ 1 hour 38 mins installing dependencies for KOLT. It uses ROS with python 3, hence the additional installation time.
+ 2 hours 27 mins installing dependencies for KOLT. (Known Object Localization and Tracking).
+ I  have installed Tensorflow, scipy, numpy, opencv-python, imgaug, scikit-learn, Keras. Now I need to 
#1/22/20
+ 1 hour 50 mins  figuring out that i need to source catkin_build_ws/install/setup.bash instead of catkin_build_ws/devel/setup.bash
 i got to the point where the realsense camera doesn't send data to the listeners. infrared works the SECOND TIME you launch ROS (and after reconnecting the camera).
Ubuntu 18.04.03 LTS
JetPack 4.3

#1/24/20
+ 3 hours 30 mins I recompiled (using wstool and catkin_make) ~/catkin_ws (workspace #1) with python3 following instructions from https://answers.ros.org/question/326226/importerror-dynamic-module-does-not-define-module-export-function-pyinit__tf2/ , downgraded setuptools to < 45 so that it would stop complaining Python2, changed tf.space_to_depth -> tf.compat.space_to_depth in   ~/catkin_build_ws/src/kolt_ros/
order of commands

	cd ~/catkin_build_ws/src/
	grep -ri tf.space_to_depth
	# (import tensorflow as tf is present in the script) go to all scripts where this occurs and replace with:
        tf.compat.space_to_depth
	(in my case the offending file was ~/catkin_build_ws/src/kolt_ros/src/core/backend.py)
	cd ~/catkin_build_ws 
	catkin build
	roslaunch src/kolt_ros/launch/k_rs_camera.launch
	# this is when I got the errors "Hardware IR configuration not supported" and "USB SCP OVERFLOW", then i hit ^c
	# ros node didn't stop. so any call to a realsense app gave 
	 24/01 16:34:37,308 WARNING [548166865344] (types.cpp:49) set_xu(ctrl=1) failed! Last Error: Resource temporarily unavailable
 	# i used rs-enumerate-devices, realsense-viewer, and rs-sensor-control
	cd ~/git/jetsonUtilities
	./jetsonInfo.py
	cd scripts/
	./jetson_variables
	cd ~/catkin_build_ws
	catkin build
	. devel/setup.bash
	. install/setup.bash
	pip install "setuptools<45"
	rosdep check kolt

 2008  sudo apt-get install ros-melodic-vision-msgs
# even though vision-msgs is a package in this workspace, I still installed it through apt-get
 2014  rosdep check kolt
# all dependencies installed
 2015  rosdep check realsense2_camera
 2016  sudo apt-get install ros-melodic-ddynamic-reconfigure
 2017  sudo apt-get install ros-melodic-librealsense2
# even though ddynamic-reconfigure is a package in this workspace, i installed it through apt-get
# i installed ros-melodic-librealsense2 previously but saw no improvement, at this point I'm just doing it to get rosdep to return "all deps installed"
 2020  roslaunch src/kolt_ros/launch/k_rs_camera.launch 
 2021  catkin build
 2022  . devel/setup.bash
 2023  . install/setup.bash
 2024  roslaunch src/kolt_ros/launch/k_rs_camera.launch
	catkin config --install
# At this point it is clear that somehow, ~/catkin_build_ws/install/share/kolt/launch isn't being created
	cd src/kolt_ros/
	vim CMakeLists.txt

# Add this to CMakeLists.txt:
install(DIRECTORY launch/
    DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch
    )
# and add python scripts to install to line 108
catkin_install_python(PROGRAMS scripts/yolo_predict.py scripts/yolo_server.py   DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

 2056  catkin build
 2077  roslaunch kolt k_rs_camera.launch 
# vision_pose crashes from python2-compiled files.
 2083  cd ~/catkin_ws
 rm -rf build
 rm -rf devel
 cd src
# remove duplicate packages installed in this workspace
 rm -rf kolt_ros
 rm -rf ros_python3_issues
 rm -rf vision_msgs
 cd ..
# follow instructions from https://answers.ros.org/question/326226/importerror-dynamic-module-does-not-define-module-export-function-pyinit__tf2/
 2092  catkin_make
 2093  . devel/setup.bash
 2094  wstool init
 2095  wstool set -y src/geometry2 --git https://github.com/ros/geometry2 -v 0.6.5
 2096  wstool up
 2097  rosdep install --from-paths src --ignore-src -y -r
 2100  catkin_make --cmake-args -DCMAKE_BUILD_TYPE=Release -DPYTHON_EXECUTABLE=/usr/bin/python3 -DPYTHON_INCLUDE_DIR=/usr/include/python3.6m -DPYTHON_LIBRARY=/usr/lib/aarch64-linux-gnu/libpython3.6m.so

 2101  cd ..
 2102  cd  catkin_build_ws/
 2103  . devel/setup.bash
 2104  . install/setup.bash
 2105  catkin build
 2108  . devel/setup.bash
 2109  . install/setup.bash
 2110  roslaunch kolt k_rs_camera.launch 
 2114  . ~/catkin_ws/devel/setup.bash
 2115  . ~/catkin_ws/install/setup.bash
 2116  roslaunch kolt k_rs_camera.launch 

+ 1 hour 34 mins writing the above account. 
