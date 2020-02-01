# 1/12/20
+ 7 hours trying to get demos to work.
# 1/13/20
1 hour still trying to get ros with realsense to work.
Intel: They actually moved it to a side project. 
 
https://support.intelrealsense.com/hc/en-us/community/posts/360033595714-D435-USB-connection-issues
Contains info about connecting realsense cameras to ethernet and the problems when using cheap usb cables

# 1/14/20
3 hours working on fixing realsense bugs (usb driver issues in ros, tried to get a torch2trt example 
# 1/15/20 
+ 7 hours 35 min 
+ 2 hours 22 min
# 1/16/20
+ 44 min 
# 1/17/20
+ 1 hour 35 min 
Figuring out how to transfer openpose params to trt_pose example. This is so that we don't have to rely on installing openpose or ros.
Note: Jetson Xavier clocked 1k+ fps running a Pytorch model optimized using torch2trt. 
+ 3 hour 50 min 
Openpose automatically pads and resizes the input image to match the resolution of the network. AND it scales the heatmaps back to the original image.
That is necessary for this situation.
Going to use lightnet to **letterbox** input (resize & pad image while preserving aspect ratio)
# 1/18/20
+ 1 hour 21 min
Timer that shuts down pipeline and closes opencv display works now.
Confirmed that images are being letterboxed. Next step is to figure out how to rescale the heatmaps and part affinity fields.
+3 hours and 9 minutes
Isaac works and I figured out how to get it to scale the images, preserve the aspect ratio, and rescale the keypoints to the original resolution


# 1/19/20
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

# 1/21/20
+ 1 hour 38 mins installing dependencies for KOLT. It uses ROS with python 3, hence the additional installation time.
+ 2 hours 27 mins installing dependencies for KOLT. (Known Object Localization and Tracking).
+ I  have installed Tensorflow, scipy, numpy, opencv-python, imgaug, scikit-learn, Keras.
# 1/22/20
+ 1 hour 50 mins  figuring out that i need to source catkin_build_ws/install/setup.bash instead of catkin_build_ws/devel/setup.bash
 i got to the point where the realsense camera doesn't send data to the listeners. infrared works the SECOND TIME you launch ROS (and after reconnecting the camera).
Ubuntu 18.04.03 LTS
JetPack 4.3

# 1/24/20
+ 3 hours 30 mins I recompiled (using wstool and catkin_make) ~/catkin_ws (workspace #1) with python3 following instructions from https://answers.ros.org/question/326226/importerror-dynamic-module-does-not-define-module-export-function-pyinit__tf2/ , downgraded setuptools to < 45 so that it would stop complaining Python2, changed tf.space_to_depth -> tf.compat.space_to_depth in   ~/catkin_build_ws/src/kolt_ros/
order of commands

	cd ~/catkin_build_ws/src/
	grep -ri tf.space_to_depth
	## (import tensorflow as tf is present in the script) go to all scripts where this occurs and replace with:
        tf.compat.space_to_depth
	(in my case the offending file was ~/catkin_build_ws/src/kolt_ros/src/core/backend.py)
	cd ~/catkin_build_ws 
	catkin build
	roslaunch src/kolt_ros/launch/k_rs_camera.launch
	## this is when I got the errors "Hardware IR configuration not supported" and "USB SCP OVERFLOW", then i hit ^c
	## ros node didn't stop. so any call to a realsense app gave 
	 24/01 16:34:37,308 WARNING [548166865344] (types.cpp:49) set_xu(ctrl=1) failed! Last Error: Resource temporarily unavailable
 	## i used rs-enumerate-devices, realsense-viewer, and rs-sensor-control
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
## even though vision-msgs is a package in this workspace, I still installed it through apt-get
 2014  rosdep check kolt
## all dependencies installed
 2015  rosdep check realsense2_camera
 2016  sudo apt-get install ros-melodic-ddynamic-reconfigure
 2017  sudo apt-get install ros-melodic-librealsense2
## even though ddynamic-reconfigure is a package in this workspace, i installed it through apt-get
## i installed ros-melodic-librealsense2 previously but saw no improvement, at this point I'm just doing it to get rosdep to return "all deps installed"
 	roslaunch src/kolt_ros/launch/k_rs_camera.launch 
 	catkin build
 	source devel/setup.bash
 	source install/setup.bash
 	roslaunch src/kolt_ros/launch/k_rs_camera.launch
	catkin config --install
## At this point it is clear that somehow, ~/catkin_build_ws/install/share/kolt/launch isn't being created
	cd src/kolt_ros/
	vim CMakeLists.txt

## Add this to CMakeLists.txt:
	install(DIRECTORY launch/
    	DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch
    	)
## and add python scripts to install to line 108
	catkin_install_python(PROGRAMS scripts/yolo_predict.py scripts/yolo_server.py   DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

 	catkin build
 	roslaunch kolt k_rs_camera.launch 
## vision_pose crashes from python2-compiled files.
 	cd ~/catkin_ws
	rm -rf build
	rm -rf devel
	cd src
## remove duplicate packages installed in this workspace
	rm -rf kolt_ros
	rm -rf ros_python3_issues
	rm -rf vision_msgs
	cd ..
## follow instructions from https://answers.ros.org/question/326226/importerror-dynamic-module-does-not-define-module-export-function-pyinit__tf2/
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

# 1/25/20
+ 1 hour 34 mins writing the above account. 
+ 2 hours 52 mins looking for weights of required tensorflow models.
	I found a tensorflow function that resizes input images and pads them to
	target size:
	 tf.resize_with_pad
	I need to find the rescaling bounding box function next.
	The trouble I encountered this session was mainly due to neural networks.
	KOLT uses tensorflow and wants the weights for the BACKBONES of specific models.  
+ 4 hours 25 mins 
	Took 2 hours until I realized keras comes with support for pretrained models. And tensorflow comes with its own keras.
	tensorflow.keras.applications and keras.applications can be used to get specific pretrained models or load the architecture of a model, then randomly
	initialize it. For example:

	model=tensorflow.keras.applications.MobileNetV2(include_top=False,input_shape=(224,224,3),weights='imagenet')

	will load the pretrained (on imagenet dataset) model for MobileNetV2 without the top layer, which is for classification. This is what YOLO (KOLT) wants, this is the feature extraction network. If you want to load your own weights, call
	
	model.load_weights('weightsfile')

	In order to not crash the system from out-of-memory stuff, it is necessary to further optimize this model. So after you load the pretrained model:

	model_path='/home/nvidia/.keras/mobile_kolt'
	model.save(model_path)
	
	from tensorflow.python.compiler.tensorrt import trt_convert as trt
	
	follow instructions from docs.nvidia.com/deeplearning/frameworks/tf-trt-user-guide/index.html#workflow-with-savedmodel

# 1/26/20
## 1 hour
	Writing log for yesterday.
## 11 hours
	Needed proper anchors for a yolo model, which would have required training.
	Note: kolt uses params, which can't be modified at runtime like args
	so instead I downloaded and implemented DEMO #4 in([link](https://github.com/jkjung-avt/tensorrt_demos/)
### Tensorrt demos
	It worked in that environment, so next I had to port it over. 
	It was completely plug-and-play
	I copied over yolov3.py and yolov3_classes and yolov3-416.trt plus some other files. I wasn't able to figure out how to add a directory that ros would pick up and put in the install directory so I just added yolov3.py to core and added it to kolt.egg-info/SOURCES.txt and added its import to src/core/__init__.py. I also refactored the code in yolo_server.py to accept inputs correctly from yolov3, added rospy.loginfo calls in yolov3 and that was it. It failed at the inference step. gives:
	[TensorRT] ERROR: ../rtSafe/cuda/reformat.cu (842) - Cuda Error in NCHWToNCHHW2: 33 (invalid resource handle)

### Realsense error fix
	Also, to get rid of the "out of frame resources" message from the realsense cameras i changed the default framerate to 15 from 30. I also disabled infrared2 by default. I didn't specify any arguments, I directly changed the launch file: realsense-ros/realsense2_camera/launch/rs_aligned_depth.launch
	At runtime i do initial_reset:=true because sometimes the camera freaks out and gives USB SEP overflow, USB SCP Overflow, Hardware Error. 
# 1/27/20
## 45 minutes
	Writing log for yesterday
## 7 hours 11 min
+ Fixed tensorrt cuda error. Started cuda context and constructed trtyolov3 at the start of every call to _handle_yolo_detect, then deleted them at the end. it works as far as getting vision_poses published, bearings calculated, and bounding boxes published, but yolo_Server crashes without warning after less than 10 calls.
# 1/29/20
+ forum post about this topic: https://devtalk.nvidia.com/default/topic/1056268/tensorrt/tensorrt-do_inference-error/1
+ implementation of asynchronous inference https://github.com/jkjung-avt/tensorrt_demos/blob/master/trt_ssd_async.py
+ link to jkjung's blog post about this https://jkjung-avt.github.io/speed-up-trt-ssd/
	
+ another instance of this issue https://devtalk.nvidia.com/default/topic/1064310/tensorrt/adding-multiple-inference-on-tensorrt-invalid-resource-handle-error-/

+ Thread safety tensorrt documentation (not extremely helpful tbh) https://docs.nvidia.com/deeplearning/sdk/tensorrt-archived/tensorrt-601/tensorrt-best-practices/index.html#thread-safety

+ I spent roughly 3 hours playing with yolo_server.py to get the inference to work without constructing the inference engine at every request to predict. It only worked when requests came from within its own context (i.e. before rospy::Service::spin() is called).

+ I modeled my incomplete solution of the examples in jkjung's repo.

# 2/1/20
+ I removed the hdmi connection from the jetson to save RAM. Without a display connected there may be a resolution issue when accessing the nano from vnc (too small) i appended this to /etc/X11/xorg.conf :
	Section "Screen"
     Identifier    "Default Screen"
     Monitor       "Configured Monitor"
     Device        "Tegra0"
     SubSection "Display"
         Depth    24
         Virtual 1280 800 # Modify the resolution by editing these values
     EndSubSection
    EndSection

+ credit to https://devtalk.nvidia.com/default/topic/1069710/jetson-nano/headless-jetson-nano-capabilities/ for the solution
	