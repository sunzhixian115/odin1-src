开驱动

source install/setup.bash

ros2 launch odin_ros_driver odin1_ros2.launch.py


保存bin地图

cd src/odin_ros_driver
./set_param.sh save_map 1 # 执行保存脚本

保存pcd地图
./save_pcd.sh



看tf树

ros2 run tf2_tools view_frames


cd src/odin_ros_driver-0.10.0 # $PATH为用户保存驱动的源目录
./set_param.sh save_map 1 # 执行保存脚本