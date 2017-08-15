---
translated_page: https://github.com/PX4/Devguide/blob/master/en/ros/external_position_estimation.md
translated_sha: 95b39d747851dd01c1fe5d36b24e59ec865e323e
---

# 使用视觉或运动捕捉系统

> **知悉：** 在学习以下教程之前，请确保你已经拥有一套能使用LPE模块的自驾仪固件。你可以在PX4最新版本的压缩包中找到PX4固件的LPE版本，或者你可以使用命令`make px4fmu-v2_lpe`从源码编译得到。更多详情参见[Building the Code](../setup/building_px4.md)。

该教程旨在使用从不同方法（除GPS之外）得到位置数据来构建一个基于PX4的系统，例如使用VICON、Optitrack以及其他基于视觉的估计系统，如[ROVIO](https://github.com/ethz-asl/rovio)、[SVO](https://github.com/uzh-rpg/rpg_svo)或者[PTAM](https://github.com/ethz-asl/ethzasl_ptam)。

位置估计既可由机载计算机发送，也可来自机外（例如：VICON）。这些数据用于更新机体相对于本地坐标系的位置估计。来自视觉/运动捕捉系统的数据也可以选择性地整合进姿态估计器中。

这个系统可以作为应用，来进行室内位置控制或者基于视觉的航点导航。

对于视觉，用于发送位姿数据的MAVLink消息是[VISION_POSITION_ESTIMATE](http://mavlink.org/messages/common#VISION_POSITION_ESTIMATE)。对于运动捕捉系统，用于发送位姿数据的MAVLink消息是[ATT_POS_MOCAP](http://mavlink.org/messages/common#ATT_POS_MOCAP)。

默认发送这些消息的应用是MAVROS的ROS-Mavlink接口。当然，也可以直接使用纯C/C++代码或者MAVLink()库来发送它们。运动捕捉系统和视觉传感器二者相应的ROS话题分别为`mocap_pose_estimate`以及`vision_pose_estimate`.更多信息可以查看[mavros_extras](http://wiki.ros.org/mavros_extras).

**以下功能仅用LPE估计器时测试过.**

## 用LPE调整视觉或运动捕捉

### 使能外部位姿输入

需要设置几个参数（从QGroundControl或者NSH shell）来使能或者禁用视觉/运动捕捉。


> 设置系统参数```CBRK_NO_VISION```为0来使能视觉位置估计。 

> 设置系统参数```ATT_EXT_HDG_M```为1或者2来使能外部朝向估计。设置为1使用视觉，设置为2使用运动捕捉。

#### 禁用气压计融合
如果视觉或运动捕捉已经提供了一个高度准确的可用高度信息，那么禁用LPE中的气压校正也许可以减少Z轴的漂移。

参数`LPE_FUSION`中有一位(1 bite)的空间是用于禁用气压计融合的，你可以从QGroundControl中设置，即，取消"fuse baro"。

#### 调节噪声参数

If your vision or mocap data is highly accurate, and you just want the estimator to track it tightly, you should reduce the standard deviation parameters, `LPE_VIS_XY` and `LPE_VIS_Z` (for vision) or `LPE_VIC_P` (for motion capture). Reducing them will cause the estimator to trust the incoming pose estimate more. You may need to set them lower than the allowed minimum and force-save.
