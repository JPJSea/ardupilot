# ArduPilot_MALLARD
This repository is to compile the custom firmware for Pixhawk4 named Ardusub and flash it onto Pixhawk 4.

# Required hardware
* Pixhawk 4 flight controller
* PC running ununtu 18.04 and ros melodic
* Joypad (ps4)
* USB cables
* Thrusters, speed controllers and batteries for testing

## 1. Flash pixhawk 4 with custom firmware - Autopilot_MALLARD
ArduPilot MALLARD (AP-M) is a customised ArduPilot firmware for MALLARD. It is built based on the current stable ArduSub release (ArduSub-4.0.3).

AP-M provides a `Custom` frame configuration adapted to the thruster allocation used on MALLARD. This frame configuration allows higher level motion command input such as `move_forward`, `turn_left`, etc. AP-M also provides two new frame configurations, i.e. `JOYSTICK_PWM_CONTROL` and `ROS_PWM_CONTROL`, which enables sending a PWM signal directly to each individual motor by pushing a joystick or publishing a ROS topic (via MAVROS), respectively. The *ROS PWM Control* frame assumes thruster allocation is dealt with within ROS.

# Hardware
For this build guide, we use a Pixhawk 4 and direct USB connection to a PC running Ubuntu.

# Build
## Install git
In case you have not yet installed *git*, run the following commands in terminal:
```
sudo apt update
sudo apt install git
```

## Clone repository
Open a terminal and `cd` to our desired root folder for the repository, then clone the main [ArduPilot MALLARD](https://github.com/EEEManchester/ArduPilot_MALLARD) repository using your preferred authentication protocol.

* HTTPS:
```
git clone --recursive https://github.com/EEEManchester/ArduPilot_MALLARD.git
```
* SSH:
```
git clone --recursive git@github.com:EEEManchester/ArduPilot_MALLARD.git
```

Make sure you log into the correct GitHub account that has access to [EEEManchester](https://github.com/EEEManchester).

## Setup environment
[A script](https://github.com/ArduPilot/ardupilot/blob/master/Tools/environment_install/install-prereqs-ubuntu.sh) is provided to automatically setup your build environment.
```
cd ArduPilot_MALLARD
Tools/environment_install/install-prereqs-ubuntu.sh -y
```
> **Note:** If the end of the command promp is not showing something similar to `echo xxx end------`, the setup is unsuccessful. If it complains about `'some-python-package' has no installation candidate`, your system is probably configured for Python3. You need to run the follow instead:
> ```
> Tools/environment_install/install-prereqs-ubuntu-py3.sh -y
> ```

Reload the path (log-out and log-in to make permanent):
```
. ~/.profile
```

## Build with WAF
Make sure you are in ardupilot folder.

> **Note:** For Python3 users, you need to replace `waf` with `waf-py3`.

If you have previously built the firmware, you may want to clean WAF first:
```
./waf clean
```

Then
```
./waf configure --board Pixhawk4
./waf sub
```

After the code finishes, you can find the new firmware `ardusub.apj` in the `build/pixhawk4/bin` directory.

## Upload
### Using WAF
There are two conditions for this to work:
1. with a direct USB connection to the Pixhawk
2. only after configuring and building with `waf` before
```
./waf --upload sub
```
### Using GCS
* [Mission Planner](https://ardupilot.org/planner/docs/common-loading-chibios-firmware-onto-pixhawk.html#uploading-as-custom-firmware)
* [QGroundControl](https://docs.qgroundcontrol.com/master/en/SetupView/Firmware.html#loading-firmware)

---

### TEST the custom firmware using QGC 

1.Install QGroundControl
QGroundControl can be installed/run on Ubuntu LTS 18.04 (and later).

Ubuntu comes with a serial modem manager that interferes with any robotics related use of a serial port (or USB serial). Before installing QGroundControl you should remove the modem manager and grant yourself permissions to access the serial port. You also need to install GStreamer in order to support video streaming.

Before installing QGroundControl for the first time:

1. On the command prompt enter:
    Add the user's name to the dialout group, not the user root even though the command is run as root:   
    ```
    sudo usermod -a -G dialout $USER
    ```  
    Remove the modem manager and grant yourself permissions to access the serial port:  
    ``` 
    sudo apt-get remove modemmanager -y
    ```  
    Install GStreamer in order to support video streaming (probably not be use in MallARD):  
    ```
    sudo apt install gstreamer1.0-plugins-bad gstreamer1.0-libav gstreamer1.0-gl -y
    ```

2. Logout and login again to enable the change to user permissions.

To install QGroundControl:  
  * Download QCG from this link [QGroundControl.AppImage](https://s3-us-west-2.amazonaws.com/qgroundcontrol/latest/QGroundControl.AppImage).  
  * Install (and run) using the terminal commands from the folder containing the .AppImage file:
    ```
    chmod +x ./QGroundControl.AppImage  
    ./QGroundControl.AppImage
    ```
    
 When QGC launches you may see an error saying 'Parameters are missing from firmware. You may be running a version of firmware which is not fully supported .....'. This is normal, just proceed.

3. Finish the sensors setup in QGroundControl - click sensors and follow procedures for accelerometer and compass - autopilot rotation set to none

2.Hardware setup for test  
Connect the hardware as the photo shown below:
![hardware setup](https://user-images.githubusercontent.com/77399327/126214195-7c65f4f2-6351-4708-9f5e-b52a294ba59a.jpg)

3.Test the motor  
* Launch QGC, using command 
```
./QGroundControl.AppImage 
```
Frame selection (For more details about thruster allocation, please check this link [README_math.md](https://github.com/EEEManchester/MallARD_Pixhwak/blob/main/README_math.md)  
ArduSub frame can be configured by setting [FRAME_CONFIG](https://www.ardusub.com/developers/full-parameter-list.html#frameconfig-frame-configuration). In addition to the built-in options, we offer two additional configurations. The Custom frame has also been modified to reflect the thruster allocation of MALLARD 003.

* Set the thrust allocation:  
Click Q![Screenshot from 2021-07-20 17-06-53](https://user-images.githubusercontent.com/77399327/126358068-e0ca4cc3-65eb-4550-873f-3a1ce5251b17.png) icon --> Vehicle Setup --> Parameters.  Using the search bar: FRAME_CONFIG. Set it to 8 in advanced settings (the number associated with the joystick PWM control thrust allocation).
You may see its parameter editing panel:  
![Screenshot from 2021-07-21 20-35-48](https://user-images.githubusercontent.com/77399327/126549189-31f30050-e76f-4249-88ec-97d6426c9de2.png)


* Make sure the motors you want to test are enabled:     
 In parameters setting, using the search bar: SERVO1_FUNCTION. Check it has been set to motor1 (sometimes displayed as 33). Do the same steps to SERVO2_FUNCTION (Motor2 or 34), SERVO3_FUNCTION (Motor3 or 35), SERVO4_FUNCTION (motor4 or 36). You may see the panel like this.  
![Screenshot from 2021-07-21 10-34-34](https://user-images.githubusercontent.com/77399327/126467201-8a5fb0f8-61a5-49fb-982b-ad57c3400842.png)

* Make sure you system ID is 255:  
In Parameters, using search bar: SYSID_MYGCS, make sure it has been set to 255 (Advanced settings).  
![Screenshot from 2021-07-21 12-07-26](https://user-images.githubusercontent.com/77399327/126479324-263afdb9-82ff-41af-8063-a9e6af0a489d.png) 
* Add a virtual joystick:  
Click Q icon -->  Application settings --> General --> tick virtual joystick ![Screenshot from 2021-07-20 17-09-21](https://user-images.githubusercontent.com/77399327/126358419-3b18b2d9-4661-4400-aa69-0b49365d3181.png)    
* Arm and using the virtual joystick to make the thruster move:  
Use JOYSTICK_PWM_CONTROL frame and when you put input on x thruster 1 should move but no others. input on y only thruster 2 moves . input on yaw only thruster 3 moves. input on z only thruster 4 moves.


# Reference
## Frame selection
ArduSub frame can be configured by setting [RAME_CONFIG](https://www.ardusub.com/developers/full-parameter-list.html#frameconfig-frame-configuration). In addition to the built-in options, we offer two additional configurations. The Custom frame has also been modified to reflect the thruster allocation of MALLARD. 

| Value | Description | Comment |
| ----- | ----- | ----- |
| 7 | Custom | Tthruster allocation can be modified by editing the matrix found in [AP_Motors6DOF.cpp](https://github.com/EEEManchester/ArduPilot_MALLARD/blob/main/libraries/AP_Motors/AP_Motors6DOF.cpp#L185) @ `case SUB_FRAME_CUSTOM` |
| 8 | Joystick PWM Control | *WIP* Each motor's PWM input is mapped to a single RC channel |
| 9 | ROS PWM Control | All motors are disabled for control via MAVLink message [COMMAND_LONG (#76)](https://mavlink.io/en/messages/common.html#COMMAND_LONG) & [AV_CMD_DO_SET_MODE (176)](https://mavlink.io/en/messages/common.html#MAV_CMD_DO_SET_MODE)|

You may use QGC to select the required frame in its parameter editing panel:
![frame_config](https://user-images.githubusercontent.com/77399327/126155642-84f4bfde-8636-4cd2-a41a-349287011d40.png)
### Original guides
In ~~unlikely~~ situation that the above instructions do not work, you may want to check out the original ArduPilot guides:

* [Developers on ArduSub Wiki](http://www.ardusub.com/developers/developers.html)
* [Setting up the Build Environment (Linux/Ubuntu)](https://ardupilot.org/dev/docs/building-setup-linux.html)
* [BUILD.md](https://github.com/ArduPilot/ardupilot/blob/master/BUILD.md)
