# Xiaomi CyberDog (Athena)

[![License](https://img.shields.io/badge/License-Apache%202.0-orange)](https://choosealicense.com/licenses/apache-2.0/)

![CyberDogDog](https://cdn.cnbj2m.fds.api.mi-img.com/cyberdog-package/packages/doc_materials/cyberdogdog.jpeg)

This is the repos project of Xiaomi CyberDog. Athena is the project code. This project contains git urls of:

- Customized LCM Messages: [athena_lcm_type](https://partner-gitlab.mioffice.cn/cyberdog/athena_lcm_type)
- Embedded Drivers: [athena_drivers_gd_bin](https://partner-gitlab.mioffice.cn/cyberdog/athena_drivers_gd_bin), [athena_drivers_st](https://partner-gitlab.mioffice.cn/cyberdog/athena_drivers_st)
- Linux Kernel of Jetson Xavier NX: [athena_kernel](https://partner-gitlab.mioffice.cn/cyberdog/athena_kernel)
- Linux Kernel & RootFS of MR813: [TBD](TBD)
- Locomotion Code (Bin Currently): [athena_locomotion_bin](https://partner-gitlab.mioffice.cn/cyberdog/athena_locomotion_bin)
- ROS 2 APPs: [athena_cyberdog](https://partner-gitlab.mioffice.cn/cyberdog/athena_cyberdog), [athena_assitant](https://partner-gitlab.mioffice.cn/cyberdog/athena_assistant), [athena_automation](https://partner-gitlab.mioffice.cn/cyberdog/athena_automation), [athena_vision](https://partner-gitlab.mioffice.cn/cyberdog/athena_vision)


## Basic Information

- The default user of CyberDog is `mi`, dafault password is `123`
- You can use a USB cable to connect to the `Download` interface, and use `ssh mi@192.168.55.1` to connect to CyberDog for internal operations

## Before You Begin
---
Before you begin we recommend you prepare:

Building your own docker with our dockerfiles.

- [Dockerfile - amd64](dockers/amd64/Dockerfile)
- [Dockerfile - aarch64](dockers/aarch64/Dockerfile)

or

Installing and configuring as below.

- `python3-vcstool`: See ROS wiki page [vcstool](http://wiki.ros.org/vcstool) and [GitHub page](https://github.com/dirk-thomas/vcstool) for more details.
- `gcc-linaro-7.3.1`: Cross compiler for Linux Kernel of Jetson Xavier NX. [Download link](https://cdn.cnbj1.fds.api.mi-img.com/build-tool/gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu.tar.xz)
- ROS 2(Foxy Fitzroy): See [ROS 2 Installation](TBD) to learn ROS 2 deployment under different environment.
- MR813 Environment: See [MR813 Environment](TBD) to learn how to deploy image for locomotion board.
- Cross Compilers: Install cross compilers for arm64 and arm32 target devices. Packages' names are `gcc-aarch64-linux-gnu` and `gcc-arm-linux-gnueabihf` in Ubuntu/Debian Linux Distributions.
- CyberDog EMMC_NVME_Image: Uncompress it into a fixed directory. Make sure you have permission to read. See [Flash your CyberDog](docs/Flashing.md) for more details.
- Cross compiling files: Files under dir [cross_config](TBD).

## Architecture

### Hardware architecture

- MCU_Board_F/M/B: Based on STM32
- ROS_Board: Based on NVIDIA Jetson Xavier NX
- Loco_Board: Based on Allwinner MR813

```mermaid
graph TB
	GyroAcc_LSM---|I2C|MCUBoard_F
	Mag_AK---|I2C|MCUBoard_F
	LED_F---|I2C|MCUBoard_F
	Ultrasonic---|I2C|MCUBoard_F
	LightSensor---|I2C|MCUBoard_F
	
	MCUBoard_M---|I2C|Optical_Flow
	MCUBoard_M---|SPI|TOF
	
	MCUBoard_B---|I2C|LED_B

	Loco_Board---|UART|GyroAcc_TDK
	Loco_Board---|SPI|SPINE_F
	Loco_Board---|SPI|SPINE_B
	
	SPINE_F===|CAN1|Motor_FL_X3
	SPINE_F===|CAN2|Motor_FR_X3
	SPINE_B===|CAN1|Motor_BL_X3
	SPINE_B===|CAN2|Motor_BR_X3
	
	Codec---|I2C&I2S|ROS_Board
	PA---|I2C&I2S|ROS_Board
	Bluetooth&Wifi---|USB|ROS_Board
	TouchBoard---|I2C|ROS_Board
	
	Realsense_D450---|USB|ROS_Board
	ROS_Board---|I2C&MIPI|OV7251_X2
	ROS_Board---|I2C&MIPI|OV13B10

  MCUBoard_F===|CAN1|ROS_Board
  MCUBoard_F===|CAN2|MCUBoard_M
  MCUBoard_F===|CAN2|MCUBoard_B
  ROS_Board===|Ethernet|Loco_Board
    
```

### Software architecture

Our software are deviced into three parts:
- Basic Software Packages(Customized Operating System and development tools)
- Robotics Applications(MCU drivers, controller code(based on MIT mini cheeath), ROS 2 applications code)
- OTA Software

```mermaid
graph LR
	AndroidAPP---|Bluetooth|ROS2_Bluetooth_Bridge
	AndroidAPP---|Wifi&Socket|OTA_Server
	AndroidAPP---|WiFi&GRPC|ROS2_GRPC_Bridge
	AndroidAPP---|WiFi&RTSP|ROS2_Live_Stream
	
	ROS2_GRPC_Bridge---DDS
	ROS2_Live_Stream---|SHM|ROS2_Camera
	ROS2_Camera---|SHM|ROS2_Vision
	ROS2_Camera---DDS
	ROS2_Vision---DDS
	OTA_Server---|CAN|MCU_Driver_LED---|CAN|ROS2_LEDServer---DDS
	OTA_Server---|CAN|MCU_Driver_Ultrasonic---|CAN|ROS2_ObstacleDetection---DDS
	OTA_Server---|CAN|MCU_Driver_TOF---|CAN|ROS2_BodyStateDetection
	OTA_Server---|CAN|MCU_Driver_Mag_AK---|CAN|ROS2_BodyStateDetection
	OTA_Server---|CAN|MCU_Driver_Gyro&Acc_LSM---|CAN|ROS2_BodyStateDetection---DDS
	OTA_Server---|CAN|MCU_Driver_OpticalFlow---|CAN|ROS2_BodyStateDetection
	OTA_Server---|CAN|MCU_Driver_LightSensor---|CAN|ROS2_LightSensor---DDS
	ROS2_AudioAssitant---DDS
	ROS2_TouchDetection---DDS
	ROS2_Realsense---DDS
	
	DDS---ROS2_BatteryManager
	DDS---ROS2_VoiceCMD
	DDS---ROS2_RemoteCMD
	DDS---ROS2_DecisionMaker
	DDS---ROS2_Localization
	DDS---ROS2_Mapping
	DDS---ROS2_Navigation
	DDS---ROS2_Tracking
	
	ROS2_DecisionMaker---|Ethernet&LCM|MIT_Ctrl
	ROS2_BatteryManager---|Ethernet&LCM|Manager
	OTA_Server---|Ethernet&Socket|Manager
	
	OTA_SHELL_S>OTA_SHELL]===|UART&YMODEM|MCU_Driver_SPINE_F
	OTA_SHELL_S>OTA_SHELL]===|UART&YMODEM|MCU_Driver_SPINE_B
	
	MIT_Ctrl---|UART&Data|MCU_Driver_Gyro&Acc_TDK
	MIT_Ctrl---|SPI&Data|MCU_Driver_SPINE_F
	MIT_Ctrl---|SPI&Data|MCU_Driver_SPINE_B
	
	MCU_Driver_SPINE_F---|CAN|MCU_Driver_Motor_FL_X3
	MCU_Driver_SPINE_F---|CAN|MCU_Driver_Motor_FR_X3
	MCU_Driver_SPINE_B---|CAN|MCU_Driver_Motor_BL_X3
	MCU_Driver_SPINE_B---|CAN|MCU_Driver_Motor_BR_X3
	
	IMU_Test&Calib---|UART&Data|MCU_Driver_Gyro&Acc_TDK
	
	Manager===|UART&YMODEM|OTA_SHELL_A>OTA_SHELL]
	Manager---|UART&Data|Power_Manager
	Manager===|UART&YMODEM|Power_Manager
	Manager===|UART&YMODEM|MCU_Driver_Gyro&Acc_TDK	
```

## Fetch & Build

### Fetch

1. Add your SSH public key to your account. Ref to [SSH Keys](https://partner-gitlab.mioffice.cn/profile/keys).
2. Try to fetch projects:

```shell
$ git clone git@partner-gitlab.mioffice.cn:cyberdog/athena_repos.git
$ cd athena_repos
$ mkdir src
$ vcs import . < cyberdog.repos
```

### Build

Please check documentation in each projects for more information.

- [athena_lcm_type](https://partner-gitlab.mioffice.cn/cyberdog/athena_lcm_type): Customized LCM Messages.
- [athena_drivers_gd_bin](https://partner-gitlab.mioffice.cn/cyberdog/athena_drivers_gd_bin): Drivers of motor, SPIne, 6 axies IMU and power manager. Based on GD32s. Release with binary. Will be opened later.
- [athena_drivers_st](https://partner-gitlab.mioffice.cn/cyberdog/athena_drivers_st): Drivers of LED, TOF, light sensor, 9 axies IMU, ultrasonic sensor and optical flow sensor. Based on STM32s.
- [athena_kernel](https://partner-gitlab.mioffice.cn/cyberdog/athena_kernel): Linux Kernel of Jetson Xavier NX.
- [athnea_tina_sdk](https://partner-gitlab.mioffice.cn/cyberdog/athena_tina_sdk): Building tools of MR813.
- [athena_locomotion_bin](https://partner-gitlab.mioffice.cn/cyberdog/athena_locomotion_bin): Locomotion controller. Based on MIT mini cheetah. Release with binary. Will be opend later.
- [athena_cyberdog](https://partner-gitlab.mioffice.cn/cyberdog/athena_cyberdog): Main ROS 2 apps project. Include all ROS 2 apps except belows.
- [athena_assitant](https://partner-gitlab.mioffice.cn/cyberdog/athena_assistant): Audio assistant ROS 2 bridge based on Xiaomi Xiaoai SDK.
- [athena_automation](https://partner-gitlab.mioffice.cn/cyberdog/athena_automation): Automation applications based on ROS 2.
- [athena_vision](https://partner-gitlab.mioffice.cn/cyberdog/athena_vision): ROS 2 bridge of human faces, gestures and bodies detection & recognition. Based on Xiaomi AI Vision SDK.

## Related Pages

- [Product](https://www.mi.com/cyberdog)
- [Discussion](https://www.xiaomi.cn/board/27817860)

## Contribute

Please refer to the [CONTRIBUTING.md](CONTRIBUTING_EN.md) for a quick read-up about what to consider if you want to contribute.
