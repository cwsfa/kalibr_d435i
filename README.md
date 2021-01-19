## D435i IMU Calibration

```
# for nvidia jetson
$ cd ~/librealsense/tools/rs-imu-calibration
$ python3 rs-imu-calibration.py

# for desktop
1. Download this file
https://github.com/IntelRealSense/librealsense/blob/master/tools/rs-imu-calibration/rs-imu-calibration.py
2. Run python script
$ python3 rs-imu-calibration.py
```

* ### Command Line Parameters

|Flag   |Description   |Default|
|-----|---|---|
|`-h `|Show help. ||
|`-i <accel_filename> [gyro_filename]`| Load previously saved results to EEPROM| |
|`-s serial_no`| calibrate device with given serial_no| calibrate first found device|
|`-g `|show graph of data before and after calibration| ||
<br>

## Install Kalibr

```
$ mkdir kalibr_ws && cd kalibr_ws && mkdir src && cd src
$ sudo apt install libv4l-dev python-igraph
$ git clone https://github.com/ethz-asl/kalibr
$ cd .. && catkin build

# Type in ~/.bashrc
alias ks='source ~/karlib_ws/devel/setup.bash'

# And git clone this repository
```

## Stereo Camera calibration
```
$ roslaunch realsense2_camera rs_camera.launch
$ rosrun topic_tools throttle messages /camera/infra1/image_rect_raw 4.0 /left
$ rosrun topic_tools throttle messages /camera/infra2/image_rect_raw 4.0 /right
$ rosbag record -O stereo_calibra_d435i.bag /left /right

# set target and run
$ cd ~/kalibr_d435i/kalibr_calibrate_cameras
$ rosrun kalibr kalibr_calibrate_cameras --bag ~/kalibr_d435i/kalibr_calibrate_cameras/stereo_calibra_d435i.bag --topics /left /right --models pinhole-radtan pinhole-radtan --target ~/kalibr_d435i/april_6x6_80x80cm.yaml --show-extraction

# if an error occurs: "Cameras are not connected through mutual observations, ....."
Add option: --approx-sync 0.04
```

## Camera-IMU calibration
```
```