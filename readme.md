# TurtleBot3 ROS2 仿真自主导航项目

## 演示视频
[点击观看完整演示] https://www.bilibili.com/video/BV17u3o6XEsn/

## 项目简介
本项目在 ROS2 Humble 环境下，使用 TurtleBot3 Burger 完成 Gazebo 仿真中的 SLAM 建图与 Navigation2 自主导航。

**核心目标**：验证从零搭建仿真环境 → 实时建图 → 保存地图 → 调用地图完成导航的完整链路。

**技术栈**：Ubuntu 22.04 / ROS2 Humble / Gazebo / TurtleBot3 / slam_toolbox / Nav2

---

# TurtleBot3 ROS2 仿真自主导航：从环境配置到参数调优全流程指南

### 一、 虚拟机图形渲染崩溃修复（核心避坑）

在 VMware 中运行 Gazebo 极易遇到 `gzclient` 崩溃（报错 `Assertion px != 0 failed`）。这是因为虚拟机无法完美兼容硬件 3D 加速。

**解决步骤：**

1. 关闭 Ubuntu 虚拟机，在 VMware 的“显示器”设置中，**取消勾选“加速 3D 图形”**。
2. 启动虚拟机，打开终端，配置强制 CPU 软件渲染的环境变量：
    
    ```bash
    echo 'export LIBGL_ALWAYS_SOFTWARE=1' >> ~/.bashrc
    echo 'export GALLIUM_DRIVER=llvmpipe' >> ~/.bashrc
    source ~/.bashrc
    ```
    
3. 重启 Gazebo，画面虽略有卡顿，但可彻底解决崩溃问题。

### 二、 异步建图全流程 (SLAM)

1. **启动仿真环境**：
    
    ```bash
    source ~/turtlebot3_ws/install/setup.sh 
    export TURTLEBOT3_MODEL=burger 
    ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
    ```
    
2. **启动 SLAM 算法**：
    
    ```bash
    source ~/turtlebot3_ws/install/setup.sh 
    ros2 launch slam_toolbox online_async_launch.py
    ```
    
3. **打开一个新的终端，启动 RViz2**
    ```
    rviz2
    ```

4. **在 RViz2 中添加必要的显示组件**
    在 RViz2 界面中，点击左下角的 **"Add"** 按钮，添加以下两个关键组件：
    1. **Map**：用于显示 SLAM 构建的栅格地图。Topic 设置为 `/map`。
    2. **LaserScan**：用于显示激光雷达的扫描数据。Topic 设置为 `/scan`。

5. **开始遥控机器人建图**
    打开新的终端，启动键盘遥控节点：
    ```bash
    source ~/turtlebot3_ws/install/setup.sh 
    export TURTLEBOT3_MODEL=burger 
    ros2 run turtlebot3_teleop teleop_keyboard
    ```
    按键盘方向键控制 TurtleBot3 移动，在 RViz2 中实时看到地图逐渐构建。

### 三、 地图保存与调用（关键衔接）

1. **保存地图**：
    确认建图完整后，打开新终端执行：
    ```bash
    ros2 run nav2_map_server map_saver_cli -f ~/turtlebot3_map
    ```
    生成两个核心文件：`turtlebot3_map.pgm` 和 `turtlebot3_map.yaml`。

### 四、 启动自主导航 (Navigation2)

1. **启动仿真环境**：
    ```bash
    source ~/turtlebot3_ws/install/setup.sh 
    export TURTLEBOT3_MODEL=burger 
    ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
    ```

2. **一键启动自主导航**：
    ```bash
    export TURTLEBOT3_MODEL=burger
    ros2 launch turtlebot3_navigation2 navigation2.launch.py map:=/home/mao/turtlebot3_map.yaml autostart:=true
    ```

3. **初始化位置**：使用 2D Pose Estimate 初始化大致位置。
4. **自主导航**：使用 Nav2 Goal 指定导航目的地和方向，观察机器人自主移动。

