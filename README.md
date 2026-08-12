<div align="center">
   
# GMR-motion_tracking of DAVID-Robot-XSEN CODE
[![IsaacSim](https://img.shields.io/badge/IsaacSim-5.1-silver.svg?style=flat-square)](https://docs.omniverse.nvidia.com/isaacsim/latest/overview.html)
[![IsaacLab](https://img.shields.io/badge/IsaacLab-2.3.1-silver.svg?style=flat-square)](https://isaac-sim.github.io/IsaacLab)
[![Python](https://img.shields.io/badge/python-3.10-blue.svg?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![Python](https://img.shields.io/badge/python-3.11-blue.svg?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![Platform](https://img.shields.io/badge/platform-linux--64-orange.svg?style=flat-square&logo=linux)](https://www.linux.org/)


</div>

<div align="center">
<img width="800"  alt="image" src="https://github.com/user-attachments/assets/5e03f11c-d6f6-439f-8fbd-b24a2a7073ae" />
</div>


# 本地显卡配置
采用的电脑为ubtun24.0版本，本地电脑安装的ROS2-jaxy版本。但进行sim to sim以及后续依赖ROS2-humble，所以需要使用到DOCKER
因为受isaacsim渲染依赖要求，并没有安装到最新的驱动。
<div align="center">
<img width="800" alt="image" src="https://github.com/user-attachments/assets/12b982b0-f0ad-478e-b948-fa59b02c5acd" />
</div>



# 进行David模型对齐以及标定
依托gmr_unt仓库

1、初始给定的david为30dof，通过注释掉需要固定的关节来实现默认fixed状态，最终实现20dof，同步对齐urdf模型，将注释的部位从revolute改为fixed。

2、配置xml模型，对齐最终安全检测（AgentController 内高频执行）的关节limit。

3、查看Robot是否处于T-pose状态，根据当前显示的关节状态进行joint角度调整实现标准T-pose状态来获取一个调整好的xml模型或者通过修改
/home/unt/pro1/gmr_unt/scripts/calibrate_bvh_xsens_to_david.py 文件中ROBOT_REFERENCE_QPOS中对应关节应该旋转的数值来进行调整。
通过下述代码打开mujoco进行模型查看，拖入xml模型，关闭重力即可------
```
conda activate gmr
python -m mujoco.viewer
```

如图所示
<div align="center">
<img width="800" alt="image" src="https://github.com/user-attachments/assets/52828d86-d1f1-4b7b-8cec-d0dd7cba2d76" />
</div>

按下键盘空格键（Space），手动调整模型joint到标准T-pose状态
<div align="center">
<img width="800" alt="image" src="https://github.com/user-attachments/assets/77209971-4f1b-40ec-9bc6-3b75cc5e601e" />
</div>

# 生成Xsens下David相对人体坐标系的offset，完成IK 配置
1、通过下面两种选择进行偏移计算操作获取配置脚本

配置脚本绝对路径为
`
/home/unt/pro1/gmr_unt/calibration_output/bvh_xsens_to_unt_david.calibrated.json
`

无mujoco显示版本
```
python scripts/calibrate_bvh_xsens_to_david.py
```

mujoco显示David进行标准T-pose展示

```
python scripts/calibrate_bvh_xsens_to_david.py --view
```

如图为David处于T-pose状态

<div align="center">
<img width="800" alt="截图 2026-08-05 10-38-30" src="https://github.com/user-attachments/assets/c982e941-4bcc-46bd-a3f2-bfe70f3bee24" />
</div>


```
conda activate gmr
cd /home/unt/pro1/gmr_unt
```
# 将所获取offset同步到bvh_xsens_to_unt_david.json完成IK 配置

该文件绝对路径为
`
/home/unt/pro1/gmr_unt/general_motion_retargeting/ik_configs/bvh_xsens_to_unt_david.json
`

1、可选择下述操作完成数据覆盖
```
cp calibration_output/bvh_xsens_to_unt_david.calibrated.json \
   general_motion_retargeting/ik_configs/bvh_xsens_to_unt_david.json
```


# 进行重定向视频画面，根据重定向中各关节的实际情况进行ik配置微调（权重和坐标）

例如：
`
"Link7_Arm_L": [

    "LeftWrist",       # 目标人体关节
    
    12.0,              # 位置跟踪权重
    
    1.0,               # 姿态跟踪权重
    
    [0.0, 0.0, 0.0],   # 位置偏移xyz，通过对照mujoco中关节坐标轴进行所需调整，其中红色为x,绿色为y，蓝色为z。
    
    [0.5075, -0.4924, -0.5075, 0.4924]  # 旋转偏移（由上述T-pose 标定计算获取，一般不需要调整）。 quaternion 

]
`

其中权重调整实际是优先考虑位置或者姿态，另一者选择牺牲一部分。


<div align="center">
<img width="800"  alt="image" src="https://github.com/user-attachments/assets/f6b08416-d9c1-461f-ab7d-6c813e9d0894" />
</div>

需要注意的点：关节穿模。

```
python scripts/xsens_bvh_to_robot.py \
  --bvh_file /home/unt/pro1/gmr_unt/assets/xsens_bvh_test/251021_04_boxing_120Hz_cm_3DsMax.bvh \
  --robot unt_david \
  --save_path /home/unt/pro1/gmr_unt/output_unt_david.pkl
```

重定向过程如图

<div align="center">
<img width="800" alt="截图 2026-08-05 10-52-20" src="https://github.com/user-attachments/assets/4f99e4e5-17ff-4bac-be81-c4c1d43f3554" />
     
<img width="800" alt="截图 2026-08-05 10-52-14" src="https://github.com/user-attachments/assets/c66e0426-b1fb-4d84-a3d2-175de6ba46b0" />
</div>



进行回放查看重定向效果

```
python scripts/vis_robot_motion.py \
  --robot unt_david \
  --robot_motion_path /home/unt/pro1/gmr_unt/output_unt_david.pkl
```
 
<div align="center">
<img width="800" alt="截图 2026-08-05 10-58-34" src="https://github.com/user-attachments/assets/62c5e8e0-0577-41ac-8bb3-689d3c72f1d0" />
</div>


#  进行pkl到csv文件格式的转换
将pkl 复制一份到./motions 输出csv转化文件到 ./motions/csv中
```
cp /home/unt/pro1/gmr_unt/output_unt_david.pkl /home/unt/pro1/gmr_unt/motions/output_unt_david.pkl
```

开始进行转换

```
python scripts/batch_gmr_pkl_to_csv.py --folder motions
```

# 进行csv到npz文件的转换

```
conda activate motion_npz
cd whole_body_tracking
python scripts/csv_to_npz_unt.py \
  --input_file /home/unt/pro1/gmr_unt/motions/csv/output_unt_david.csv \
  --input_fps 30 \
  --output_fps 50 \
  --output_name unt_david_20dof \
  --output_file /home/unt/pro1/generated_motions/unt_david_20dof.npz
  --stand_duration 1.0 \
  --transition_duration 1.0 \
```
--motion_start_idx 3 \

<div align="center">
<img width="800" alt="image" src="https://github.com/user-attachments/assets/32fd9ea8-7f55-4f63-bdf7-78ae3f294b95" />
</div>

进行npz格式下在isaacsim回放测试

```
python scripts/replay_npz_unt.py \
  --motion_file /home/unt/pro1/generated_motions/unt_david_20dof.npz
```

<div align="center">
<img width="800" alt="image" src="https://github.com/user-attachments/assets/28934bd1-2767-43e9-b6fd-7dc770ab9dd7" />
</div>



上述完成整个GMR的过程。



# 基于Beyondmimic进行motion-tracking


1、在/home/unt/pro1/whole_body_tracking/source/whole_body_tracking/whole_body_tracking/robots/unt_robot.py 中定义 UNT / David 20DoF 人形机器人在 IsaacLab 里的机器人配置

2、修改/home/unt/pro1/whole_body_tracking/source/whole_body_tracking/whole_body_tracking/tasks/tracking/config/unt/flat_env_cfg.py中继承的TrackingEnvCfg的配置，去掉actor中现实难以获取的observations 

--motion_anchor_pos_b     

-- base_lin_vel

调整observation中joint_vel 的noise为n_min=-1.5, n_max=1.5---


3、整个env配置完成，采用当前本地的算法。

4、将整个环境打包送入服务器进行训练

# 服务器进行训练准备：首先配置安装IsaacLab-2.3.1以及isaacsim5.1


1、 在本地终端执行，下述代码将环境文件以及npz打包上传到服务器

```
SERVER=xxp@10.193.128.35

# 同步 whole_body_tracking 源码
rsync -avh --progress --itemize-changes \
  --exclude='__pycache__/' \
  --exclude='*.pyc' \
  /home/unt/pro1/whole_body_tracking/source/ \
  "${SERVER}:/home/xxp/data/whole_body_tracking/source/"

# 同步 motion 数据
rsync -avh --progress \
  /home/unt/pro1/generated_motions/unt_david_20dof.npz \
  "${SERVER}:/home/xxp/data/unt_david.npz"
```


# 开始服务器上的训练

1、第一次训练可以先建立临时空间
```
cd /home/xxp/data/whole_body_tracking
mkdir -p /home/xxp/data/tmp
```
2、开始训练
```
export CUDA_VISIBLE_DEVICES=0
export TMPDIR=/home/xxp/data/tmp
export TEMP=/home/xxp/data/tmp
export TMP=/home/xxp/data/tmp
export HYDRA_FULL_ERROR=1
/home/xxp/data/IsaacLab-2.3.1/isaaclab.sh -p \
scripts/rsl_rl/train_local.py \
  --task Tracking-Flat-UNT-v0 \
  --motion_file /home/xxp/data/unt_david.npz \
  --device cuda:0 \
  --headless \
  --logger tensorboard \
  --run_name unt20dof_server_train \
  --num_envs 4096
```



# 本地终端：连服务器并转发,查看实时各奖励收敛情况
```
ssh -L 6006:localhost:6007 xxp@10.193.128.35
```
# 服务器上
```
tensorboard --logdir /home/xxp/data/xxx/runs --port 6007
```

# 训练完成从服务器获取训练目录

```
scp -r xxp@10.193.128.35:/home/xxp/data/whole_body_tracking/logs/rsl_rl/unt_flat/2026-08-07_13-43-05_unt20dofwithsafe /home/unt/pro1/whole_body_tracking/logs/rsl_rl/unt_flat/
```



# 先在isaacsim下生成policy.pt

```
RUN="2026-08-04_18-18-48_unt20dof_server_train"
mkdir -p /home/unt/pro1/whole_body_tracking/logs/rsl_rl/unt_flat
cp -a \
  "/home/unt/pro1/server_training/$RUN" \
  /home/unt/pro1/whole_body_tracking/logs/rsl_rl/unt_flat/
```
# 回到本地查看训练效果
```
conda activate motion_npz
python scripts/rsl_rl/play.py   --task Tracking-Flat-UNT-v0   --num_envs 1   --load_run 2026-08-04_18-18-48_unt20dof_server_train --checkpoint model_29999.pt   --motion_file /home/unt/pro1/generated_motions/unt_david_20dof.npz
```

<div align="center">
<img width="800" alt="image" src="https://github.com/user-attachments/assets/c07fc247-c521-41ec-9387-67acb7432d03" />
</div>

# 基于mujoco进行 sim to sim验证模型的物理泛化性

首先安装完docker,进行启动
```
cd /home/unt/pro1/unt_humanoid/docker/desktop
./start.sh
```


```
conda deactivate

# 2. 加载环境
source /opt/ros/humble/setup.bash
source build/conan/conanrosenv.sh
source install/local_setup.bash   # 注意：用 local_setup，不要用 setup.bash

# 3. 编译 Debug 版
colcon build --merge-install \
  --packages-select rl_deployment \
  --cmake-args -DCMAKE_BUILD_TYPE=Debug \
  -DPython3_EXECUTABLE=/usr/bin/python3 \
  -DCMAKE_PREFIX_PATH=$(pwd)/build/conan/
```


每次更新后
```
colcon build --merge-install --packages-select rl_deployment
```


在install中rl deploy调整config mimic.yaml中的onnx路径以及帧数 以及kp和kd参数action scale
```
cd ~/pro1/unt_humanoid/docker/desktop
./start.sh
```

创建5个分屏
```
tmux new-session -s mimic -d
tmux split-window -h
tmux split-window -v
tmux select-pane -t 0
tmux split-window -v
tmux select-pane -t 2
tmux split-window -v
tmux select-layout tiled
tmux attach -t mimic
```

对每个分屏进行
```
cd ~/pro1/unt_humanoid
source install/setup.bash
source scripts/local_pc/env.sh mujoco
```
 
5个屏分别执行
0
```
ros2 run david david
```
1
```
ros2 launch david robot_state_publisher.launch.py
```
2
```
ros2 launch control control.launch.py
```
3
```
ros2 launch rl_deployment agent.launch.py
```
4
而是 Docker 里的 MuJoCo 默认没有走 NVIDIA RTX 5060，而是退化成了 CPU 软件渲染
```
__NV_PRIME_RENDER_OFFLOAD=1 \
__GLX_VENDOR_LIBRARY_NAME=nvidia \
ros2 launch mujoco_simulator mujoco.launch.py
```

如图为FSM11的状态为原地站立
<div align="center">
<img width="800"  alt="image" src="https://github.com/user-attachments/assets/f62d57ec-9e5f-486d-81e5-59d88c067728" />
</div>


另开一个终端，进行mimic的sim to sim
```
cd ~/pro1/unt_humanoid/docker/desktop
./start.sh
cd ~/pro1/unt_humanoid
source install/setup.bash
source scripts/local_pc/env.sh mujoco
ros2 topic pub --once /rl/fsm_switch std_msgs/msg/Int32 "{data: 26}"
```
效果如图
<div align="center">
<img width="800"  alt="image" src="https://github.com/user-attachments/assets/3b4f20be-a765-48a8-b07e-a3ed2960c83c" />
</div>

```
cd ~/pro1/unt_humanoid
source install/setup.bash
source scripts/local_pc/env.sh mujoco
```
```
ros2 topic pub --once /rl/fsm_switch std_msgs/msg/Int32 "{data: 26}"
```





rsync -avhP -e "ssh -p 222" /home/unt/pro1/unt_humanoid/install/ tztek@10.193.252.21:/agibot/data/home/tztek/xxp/install/


ssh -p 222 tztek@10.193.252.21




