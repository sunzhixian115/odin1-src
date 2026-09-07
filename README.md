把odin1他tf树方向改了：直接在驱动 TF 发布源头把 odom→map 改为 map→odom，同时对位姿求逆：旋转矩阵取转置、平移变为 -Rᵀt、四元数取共轭。不额外发布反向 TF，而是直接替换原来的 TF，因此树中只保留 map→odom→base_link，避免形成环。





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
