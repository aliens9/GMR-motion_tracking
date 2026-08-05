# 首先安装IsaacLab-2.3.1以及isaacsim5.1

#  XSEN光捕重定向         GMR-motion_tracking

查看是否处于T-pose状态，根据实际情况进行调整。

`conda activate gmr`


# 仅生成 offset 和 IK 配置

`python scripts/calibrate_bvh_xsens_to_david.py`
# 或标定后顺便看 T-pose

`python scripts/calibrate_bvh_xsens_to_david.py --view`

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

`python scripts/batch_gmr_pkl_to_csv.py --folder pkl_export`




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







