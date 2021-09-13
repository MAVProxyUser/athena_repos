# CyberDog-DDS本地及多播设置

**涉及配置文件**

  1. 开机启动service文件：
  - /etc/systemd/system/bluetooth_ros2.service
  - /etc/systemd/system/cyberdog_ros2.service
  - /etc/systemd/system/cyberdog_automation.service
  2. dds配置xml文件
  - /etc/systemd/system/cyclonedds.xml
  3. config文件
  - /etc/mi/mi_config

**更改项目**

  1. service文件：
  - 添加ROS_LOCALHOST_ONLY=1环境变量，使ROS消息本地化
  - 添加CYCLONEDDS_URI=环境变量，加载cyclonedds.xml，使DDS消息本地化

```
[Unit]
Description="cyberdog Deamon"

[Service]
User=mi
Type=idle
Environment="ROS_DOMAIN_ID=42"
Environment="ROS_VERSION=2"
Environment="ROS_PYTHON_VERSION=3"
Environment="ROS_DISTRO=foxy"
Environment="LD_LIBRARY_PATH=/opt/ros2/foxy/lib:/opt/ros2/cyberdog/lib"
Environment="PYTHONPATH=/opt/ros2/foxy/lib/python3.6/site-packages:/opt/ros2/cyberdog/lib/python3.6/site-packages"
Environment="AMENT_PREFIX_PATH=/opt/ros2/foxy:/opt/ros2/cyberdog"
Environment="DISPLAY=:0"
Environment="RMW_IMPLEMENTATION=rmw_cyclonedds_cpp"
Environment="ROS_LOCALHOST_ONLY=1"
Environment="CYCLONEDDS_URI=file:///etc/systemd/system/cyclonedds.xml"

ExecStart=/opt/ros2/foxy/bin/ros2 launch athena_bringup lc_bringup_launch.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

  2. cyclonedds.xml配置文件

参考[官网配置文档](https://github.com/eclipse-cyclonedds/cyclonedds/blob/master/docs/manual/config.rst)
  - 配置General/NetworkInterfaceAddress为lo本地
  - 配置General/AllowMulticast为false，关闭多播
  - 配置Discovery/Peers为<Peer address="localhost"/>本地

```XML
<?xml version="1.0" encoding="UTF-8" ?>
<CycloneDDS xmlns="https://cdds.io/config" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"    xsi:schemaLocation="https://cdds.io/config https://raw.githubusercontent.com/eclipse-cyclonedds/cyclonedds/master/etc/cyclonedds.xsd">
    <Domain id="42">
        <General>
            <NetworkInterfaceAddress>lo</NetworkInterfaceAddress>
            <AllowMulticast>false</AllowMulticast>
        </General>
        <Discovery>
            <ParticipantIndex>auto</ParticipantIndex>
            <MaxAutoParticipantIndex>30</MaxAutoParticipantIndex>
            <Peers>
                <Peer address="localhost"/>
            </Peers>
        </Discovery>
    </Domain>
</CycloneDDS>
```

  3. mi_config配置文件
  - 添加ROS_LOCALHOST_ONLY=1环境变量，使ROS消息本地化
  - 添加CYCLONEDDS_URI=环境变量，加载cyclonedds.xml，使DDS消息本地化

```Bash
export ROS_DOMAIN_ID=42
export ROS_VERSION=2
export ROS_PYTHON_VERSION=3
export ROS_DISTRO=foxy
source /opt/ros2/foxy/local_setup.bash
source /opt/ros2/dobot/setup.bash
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export ROS_LOCALHOST_ONLY=1
export CYCLONEDDS_URI=file:///etc/systemd/system/cyclonedds.xml
```

**快捷恢复默认网络配置**

***注意：恢复后切换网络环境可能出现UDP消息传输失败的情况***

注释掉所有service文件以及mi_config中消息本地化的配置环境：
  - ROS_LOCALHOST_ONLY=1
  - CYCLONEDDS_URI=file:///etc/systemd/system/cyclonedds.xml

**多播设置**

请参照官方配置文档，按需配置cyclonedds.xml