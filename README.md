# Sensor-communication-protocol
介绍机器人中常见传感器的通信协议与配置

### 1.雷达(以noetic配置mid360为例)
#### 前置网络配置知识
[虚拟机桥接mid360雷达教程](https://blog.csdn.net/zaiiazqqq/article/details/146409439?spm=1001.2014.3001.5501) 

Q:为什么要使用有线网络传输雷达数据 \
雷达产生的原始数据量极大，以太网协议对应OSI模型的数据链路层,提供高达1Gbps的带宽,远超传统串口或CAN总线的能力,可满足实时传输需求,避免数据延迟或丢失 \
网线作为物理层用于传输数据,比无线网络更稳定更快 \
Q:虚拟机连接雷达 \
虚拟机连接雷达需要与雷达处于同一局域网下才能实现数据传输---主机、虚拟机的网络位与雷达相同 \
方式:\
主机配置以太网IP和子网掩码;虚拟机设置桥接,配置IP与子网掩码,修改livox_ros_driver2中config和launch文件中对应参数;\
雷达IP是192.168.1.1xx(xx是S/N码最后两位)\
例:mid360的S/N码为47xxxx40020122,那么雷达IP是192.168.1.122,主机与虚拟机的IP需要设置为192.168.1.xxx(三者IP不能相同),子网掩码设置为255.255.255.0 \

#### 雷达ROS包配置
| **组件**         | **层级定位**       | **输入/输出**              | **依赖关系**                     |  
|------------------|-------------------|---------------------------|----------------------------------|  
| Livox-SDK2       | 硬件驱动层         | 激光雷达原始数据 → 结构化数据 | 无（直接与硬件交互）              |  
| livox_ros_driver2 | ROS中间件层       | 结构化数据 → ROS消息        | 依赖Livox-SDK2          |  
| FAST-LIO         | 算法应用层         | ROS消息 → 位姿与地图        | 依赖livox_ros_driver2   |  


a.[Livox-SDK2源码](https://github.com/Livox-SDK/Livox-SDK2) \

底层组件,按照官方文档sudo make install后,API被安装在系统目录下,livox_ros_driver2可直接调用 \
```
$ git clone https://github.com/Livox-SDK/Livox-SDK2.git
$ cd ./Livox-SDK2/
$ mkdir build &&  cd build
$ cmake .. && make -j
$ sudo make install
```

b.[livox_ros_driver2源码](https://github.com/Livox-SDK/livox_ros_driver2) \

```
(文件传输可能build运行不起来,需要直接从github拉取代码)
git clone https://github.com/Livox-SDK/livox_ros_driver2.git ws_livox/src/livox_ros_driver2
source /opt/ros/noetic/setup.bash
cd ./ws_livox/src/livox_ros_driver2
./build.sh ROS1
```
```
修改livox_ros_driver2下的config文件和launch文件,S/N码、雷达IP、虚拟机IP
```
```
# 尝试运行,验证配置
$ source ./ws_livox/devel/setup.bash
$ roslaunch livox_ros_driver2 rviz_MID360.launch
```
在rviz中的style选项修改points为squares后,出现点云图像 \

c.配置FAST-LIO \

```
sudo apt install libeigen3-dev
sudo apt install libpcl-dev
```
```
# 进入刚才的ws_livox工作空间下,也可以新建工作空间
git clone https://github.com/hku-mars/FAST_LIO.git
cd FAST_LIO
git submodule update --init
# 替换xml,cmake,src文件里的livox_ros_driver为livox_ros_driver2再编译
cd ../..
catkin_make
```
```
# 进入livox_ros_driver2所在的工作空间
source devel/setup.bash
roslaunch livox_ros_driver2 msg_MID360.launch
# 再开一个终端，进入fast_lio所在的工作空间
source devel/setup.bash
roslaunch fast_lio mapping_mid360.launch
```



