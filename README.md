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
<div align="center">
<img width="800" alt="image" src="https://github.com/user-attachments/assets/12b982b0-f0ad-478e-b948-fa59b02c5acd" />
</div>



# 查看Robot是否处于T-pose状态，根据实际情况进行调整。


配置xml模型，对齐最终安全检测（AgentController 内高频执行）的关节limit。
```
conda activate gmr
python -m mujoco.viewer
```
拖入xml模型，关闭重力即可

<div align="center">
<img width="800" alt="image" src="https://github.com/user-attachments/assets/52828d86-d1f1-4b7b-8cec-d0dd7cba2d76" />
</div>

按下键盘空格键（Space），手动调整模型joint到标准T-pose状态
<div align="center">
<img width="800" alt="image" src="https://github.com/user-attachments/assets/77209971-4f1b-40ec-9bc6-3b75cc5e601e" />
</div>
# 仅生成 offset 和 IK 配置

```
python scripts/calibrate_bvh_xsens_to_david.py
```

# 或标定后顺便看 T-pose

```
python scripts/calibrate_bvh_xsens_to_david.py --view
```

如图为T-pose状态
<div align="center">
<img width="800" alt="截图 2026-08-05 10-38-30" src="https://github.com/user-attachments/assets/c982e941-4bcc-46bd-a3f2-bfe70f3bee24" />
</div>


# 获取偏移 ,在文档
general_motion_retargeting/ik_configs/bvh_xsens_to_unt_david.json


```
conda activate gmr
cd /home/unt/pro1/gmr_unt
```
# 同步 IK 配置
```
cp calibration_output/bvh_xsens_to_unt_david.calibrated.json \
   general_motion_retargeting/ik_configs/bvh_xsens_to_unt_david.json
```


# 进行重定向，根据重定向中的实际情况进行ik配置微调（权重和坐标）
使用文件绝对路径
```
python scripts/xsens_bvh_to_robot.py \
  --bvh_file /home/unt/pro1/gmr_unt/assets/xsens_bvh_test/251021_04_boxing_120Hz_cm_3DsMax.bvh \
  --robot unt_david \
  --save_path /home/unt/pro1/gmr_unt/output_unt_david.pkl
```

重定向如图
<div align="center">
<img width="800" alt="截图 2026-08-05 10-52-20" src="https://github.com/user-attachments/assets/4f99e4e5-17ff-4bac-be81-c4c1d43f3554" />
     
<img width="800" alt="截图 2026-08-05 10-52-14" src="https://github.com/user-attachments/assets/c66e0426-b1fb-4d84-a3d2-175de6ba46b0" />
</div>



进行回放查看效果
```
python scripts/vis_robot_motion.py \
  --robot unt_david \
  --robot_motion_path /home/unt/pro1/gmr_unt/output_unt_david.pkl
```
 
<div align="center">
<img width="800" alt="截图 2026-08-05 10-58-34" src="https://github.com/user-attachments/assets/62c5e8e0-0577-41ac-8bb3-689d3c72f1d0" />
</div>


# 进行pkl到csv文件格式的转换
将pkl 复制一份到./motions 输出csv转化文件到 ./motions/csv中

```
python scripts/batch_gmr_pkl_to_csv.py --folder pkl_export
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
```
<div align="center">
<img width="800" alt="image" src="https://github.com/user-attachments/assets/32fd9ea8-7f55-4f63-bdf7-78ae3f294b95" />
</div>

进行回放测试

```
python scripts/replay_npz_unt.py \
  --motion_file /home/unt/pro1/generated_motions/unt_david_20dof.npz
```

<div align="center">
<img width="800" alt="image" src="https://github.com/user-attachments/assets/28934bd1-2767-43e9-b6fd-7dc770ab9dd7" />
</div>



# 在本地终端执行上传到服务器：

```
rsync -avh --progress --itemize-changes \
  --exclude='__pycache__/' \
  --exclude='*.pyc' \
  /home/unt/pro1/whole_body_tracking/source/ \
  xxp@10.193.128.35:/home/xxp/data/whole_body_tracking/source/
同步脚本目录：
rsync -avh --progress \
  /home/unt/pro1/generated_motions/unt_david_20dof.npz \
  "${SERVER}:/home/xxp/data/unt_david.npz"
```


# 开始服务器上的训练

#安装IsaacLab-2.3.1以及isaacsim5.1

```
cd /home/xxp/data/whole_body_tracking
mkdir -p /home/xxp/data/tmp
```

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



# 训练完成从服务器获取训练目录

```
rsync -avh --progress   xxp@10.193.128.35:/home/xxp/data/whole_body_tracking/logs/rsl_rl/unt_flat/2026-08-04_18-18-48_unt20dof_server_train/   /home/unt/pro1/server_training/2026-08-04_18-18-48_unt20dof_server_train/
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
