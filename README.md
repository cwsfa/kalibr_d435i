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
alias ks='source ~/kalibr_ws/devel/setup.bash'

# And git clone this repository
```

## Stereo Camera calibration
```
$ roslaunch realsense2_camera rs_camera.launch
$ rosrun topic_tools throttle messages /camera/infra1/image_rect_raw 4.0 /left
$ rosrun topic_tools throttle messages /camera/infra2/image_rect_raw 4.0 /right
$ rosbag record -O stereo_calibra_d435i.bag /left /right

# set target and run (If the target board size is A4, change april_6x6_80x80cm.yaml to april_6x6_A4.yaml)
$ cd ~/kalibr_d435i/kalibr_calibrate_cameras
$ rosrun kalibr kalibr_calibrate_cameras --bag ~/kalibr_d435i/calibrate_cameras/stereo_calibra_d435i.bag --topics /left /right --models pinhole-radtan pinhole-radtan --target ~/kalibr_d435i/april_6x6_80x80cm.yaml --show-extraction

# if an error occurs: "Cameras are not connected through mutual observations, ....."
Add option: --approx-sync 0.04
```

## Get noise density and bias random walk through imu_utils package
- Preparation and Collection
```
1. Install required libary
$ sudo apt install libdw-dev libgoogle-glog-dev libatlas-base-dev

2. Install package in ~/calib_ws/src
$ git clone https://github.com/gaowenliang/code_utils.git
$ cd .. && catkin build code_utils
$ cd src && git clone https://github.com/gaowenliang/imu_utils.git
$ cd .. && catkin build imu_utils

3. Move imu_utils_d435i.launch to imu_utils launch dir and start launch file
$ roslaunch imu_utils imu_utils_d435i.launch

4. Collect the data while the IMU is Stationary, with set duration time
```
- Create imu_d435i.yaml file with results
    - result 
    ```
    type : IMU
    name : d435i
    Gyr :
    unit : "rad/s".
    avg-axis :
        gyr_n : 3.0247286551398204e-03
        gyr_w : 2.2901170839457266e-05
    x-axis :
        gyr_n : 3.0486121293259861e-03
        gyr_w : 2.3588043857742811e-05
    y-axis :
        gyr_n : 3.9906956275203298e-03
        gyr_w : 3.8510954801834231e-05
    z-axis :
        gyr_n : 2.0348782085731448e-03
        gyr_w : 6.6045138587947500e-06
    Acc :
    unit : "m/s^2".
    avg-axis :
        acc_n : 2.8949600741350223e-02
        acc_w : 4.5552273079824281e-04
    x-axis :
        acc_n : 2.4870413946165581e-02
        acc_w : 3.5175805157083910e-04
    y-axis :
        acc_n : 2.7808521377206314e-02
        acc_w : 4.8805723243924246e-04
    z-axis :
        acc_n : 3.4169866900678782e-02
        acc_w : 5.2675290838464703e-04
    ```
    - Create imu_d435i.yaml using the results above
    ```
    rostopic: /camera/imu
    update_rate: 200.0 # Hz
    
    accelerometer_noise_density: 2.89e-01
    accelerometer_random_walk: 4.55e-04 
    gyroscope_noise_density: 3.02e-03
    gyroscope_random_walk: 2.29e-05
    ```


## Camera-IMU calibration
```
$ cd ~/kalibr_d435i/calibrate_imu_camera

# If the target board size is A4, change april_6x6_80x80cm.yaml to april_6x6_A4.yaml

## Method1
$ rosbag record -O stereo_imu_calibra_d435i.bag /camera/infra1/image_rect_raw /camera/infra2/image_rect_raw /camera/imu
$ rosrun kalibr_calibrate_imu_camera --target ~/kalibr_d435i/april_6x6_80x80cm.yaml --cam ~/kalibr_d435i/calibrate_cameras/calibra_d435i.yaml --imu ~/kalibr_d435i/calibrate_imu_camera/imu_d435i.yaml --bag ~/kalibr_d435i/calibrate_imu_camera/stereo_imu_calibra_d435i.bag

# Method2 (better??)
$ rosrun topic_tools throttle messages /camera/infra1/image_rect_raw 20.0 /left
$ rosrun topic_tools throttle messages /camera/infra2/image_rect_raw 20.0 /right
$ rosrun topic_tools throttle messages /camera/imu 200.0 /imu
$ rosbag record -O imu_stereo_d435i_20_200hz.bag  /left /right /imu
#### change rostopic value of imu_d435i.yaml to /imu
$ rosrun kalibr_calibrate_imu_camera --target ~/kalibr_d435i/april_6x6_80x80cm.yaml --cam ~/kalibr_d435i/calibrate_cameras/calibra_d435i.yaml --imu ~/kalibr_d435i/calibrate_imu_camera/imu_d435i.yaml --bag ~/kalibr_d435i/calibrate_imu_camera/imu_stereo_d435i_20_200hz.bag
```

## Make VINS-Fusion config file
- camchain-imucam.yaml

| d435i_stereo_imu.yaml | camchain-imucam.yaml | d435i_imu_param.yaml |
|-----|---|---|
|body_T_cam0|cam0:T_cam_imu||
|body_T_cam1|cam1:T_cam_imu||
|acc_n||acc_n|
|gyr_n||gyr_n|
|acc_w||acc_w|
|gyr_w||gyr_w|

- left.yaml, right.yaml

| left.yaml & right.yaml | d435i_imu_param.yaml |
|-----|---|
|distortion_parameters|distortion_coeffs|
|projection_parameters|intrinsics|

## Run VINS-Fusion
```
$ roslaunch realsense2_camera rs_camera.launch
$ roslaunch vins vins_rviz.launch
$ rosrun vins vins_node ~/kalibr_d435i/calibrate_imu_camera/vins_config/d435i_stereo_imu.yaml 
$ rosrun loop_fusion loop_fusion_node ~/kalibr_d435i/calibrate_imu_camera/vins_config/d435i_stereo_imu.yaml 
```