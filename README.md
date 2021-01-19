## Install Kalibr

```
cd catkin_ws/src/
sudo apt install libv4l-dev python-igraph
git clone https://github.com/ethz-asl/kalibr
cd .. && catkin build
```
<br>

## Stereo Camera calibration
```
roslaunch realsense2_camera rs_camera.launch
rosrun topic_tools throttle messages /camera/infra1/image_rect_raw 4.0 /left
rosrun topic_tools throttle messages /camera/infra2/image_rect_raw 4.0 /right
rosbag record -O stereo_calibra_d435i.bag /left /right

# set target and run
rosrun kalibr kalibr_calibrate_cameras --bag /home/shin/kalibr/stereo_calibra_d435i.bag --topics /left /right --models pinhole-radtan pinhole-radtan --target /home/shin/kalibr/april_6x6_80x80cm.yaml

or

# if an error occurs: "Cameras are not connected through mutual observations, ....."
rosrun kalibr kalibr_calibrate_cameras --bag /home/shin/kalibr/stereo_calibra_d435i.bag --topics /left /right --models pinhole-equi pinhole-equi --target /home/shin/kalibr/april_6x6_80x80cm.yaml --show-extraction --approx-sync 0.04

```
<br>

## Camera-IMU calibration
```
```