# 如何线刷铁蛋

约定：铁蛋为Target端，刷机用主机是Host端。

## 环境准备

### 1. 安装烧录相关的环境

推荐使用Ubuntu 18.04 / Ubuntu 20.04，或对应版本的Debian操作系统。

安装流程：

```
sudo apt-get install device-tree-compiler
sudo apt-get install nfs-common
sudo apt-get install sshpass
sudo apt-get install abootimg
sudo apt-get install network-manager
sudo apt-get install libxml2-utils
```
### 2. 确定具备下载线，和持续供电的铁蛋

1. 下载线可以是普通的USB线，也可以是附送的刷机线（应该是黑色的），二者的区别是，通过刷机线启动，设备可以自动进入刷机模式，而前者则需要手动触发刷机模式。类似于NV Jetson平台的`Recovery Pin`接地的操作。

2. 保证铁蛋持续供电，以确保刷机的完整执行。

## 进入刷机模式

### 方法1：直接使用刷机线

刷机线如上述介绍，确保`先接线`，`再上电`。才能保证设备正确进入刷机模式。

### 方法2：通过任意USB线

通过任意USB线连接`DOWNLOAD`接口，确保已经开机，并通过下面指令ssh连接设备。密码为123。

```
ssh mi@192.168.55.1
```

连接成功后，可以通过

```
sudo reboot --force forced-recovery
```

进入刷机模式。

### 验证刷机模式

通过在Host端（刷机端）输入`lsusb`，寻找`Nvidia Corp.`相关字样。如存在，则代表进入刷机模式成功。

## 烧录镜像

### 1. 下载镜像

将镜像下载到Host端。

```
wget https://cdn.cnbj2m.fds.api.mi-img.com/cyberdog-package/build/athena_foxy_2021.08.24_emmc_nvme_V1.0.0.66.20210824_release_bbcc37a86a.tgz
```

### 2. 解压

```
tar -xf athena_foxy_2021.08.24_emmc_nvme*
```

### 3. 刷机

```
sudo systemctl stop udisks2.service
sudo ./flashall.sh
```

### 4. 等待

等到刷完