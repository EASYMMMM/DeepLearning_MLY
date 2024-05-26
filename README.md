# 深度学习 大作业

孟令一

## 1. 文件结构

- 课程报告PDF。

- 文件夹`IGEvolution`为实验源码。使用IsaacGym进行实验。该文件夹内包含了[IsaacGym](https://github.com/NVIDIA-Omniverse/IsaacGymEnvs)的所有代码，且自己编写的代码也在其中。

- 两个结果演示视频。



## 2. 环境配置

[Isaac Gym环境安装和四足机器人模型的训练-CSDN博客](https://blog.csdn.net/weixin_44061195/article/details/131830133?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2~default~YuanLiJiHua~Position-2-131830133-blog-124605383.235^v38^pc_relevant_sort&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~YuanLiJiHua~Position-2-131830133-blog-124605383.235^v38^pc_relevant_sort&utm_relevant_index=5)

在官网下载最新的文件包[Isaac Gym - Preview Release](https://developer.nvidia.com/isaac-gym)，注意需要登陆。

```
# 在文件根目录里运行
./create_conda_env_rlgpu.sh
# 激活环境
conda activate rlgpu
# 正常情况下这时候可以运行实例程序/isaacgym/python/examples/joint_monkey.py
python joint_monkey.py
# 可以通过--asset_id命令控制显示的模型
python joint_monkey.py --asset_id=6

```

到这里IsaacGym仿真环境就安装好了。

`./create_conda_env_rlgpu.sh`要等很久是正常的

出现这样的报错：

`
ImportError: libpython3.7m.so.1.0: cannot open shared object file: No such file or directory`

需要`sudo apt install libpython3.7`

或者`export LD_LIBRARY_PATH=/home/ps/anaconda3/envs/rlgpu/lib`

大概率后者设置路径有效。

**还需要进行IsaacGym基础训练环境安装：**

[GitHub](https://github.com/NVIDIA-Omniverse/IsaacGymEnvs)上下载好 进文件内

```
conda activate rlgpu
pip install -e .
```

到train.py 的文件夹路径内训练

```
python train.py task=Ant
# 不显示动画只训练
python train.py task=Ant headless=True
# 测试训练模型的效果，num_envs是同时进行训练的模型数量
python train.py task=Ant checkpoint=runs/Ant/nn/Ant.pth test=True num_envs=64

```

## 3. 代码结构

以下为代码文件夹中的内容。仅展示了自己编写的代码。

```bash
IGEvolution
│   ...
└───tensorboard          # 存储实验结果
│  └─────...
└───isaacgymenvs
│  │  ...
│  │  SRL_Evo_train.py   # 主文件，进行训练和演示
│  └─────...
│  └─────runs           # 存储训练结果
│  │  │ AMP_run_04-21-12-09.pth
│  │  └ SRL_walk_26-14-57-33.pth
│  └─────cfg            
│  │  └─────pbt
│  │  └─────task
│  │  │    │     ...   
│  │  │    └─ HumanoidAMPSRLTest.yaml  # 环境配置文件
│  │  └─────train
│  │     │     ...   
│  │     └───  HumanoidAMPSRLTestPPO.yaml  # PPO训练配置文件
│  └─────learning
│  │  └───  ...
│  │  └─────SRLEvo     # 外肢体（进化）
│  │       │  srl_continuous.py       # 顶层 与env交互+采样+使用PPO算法训练
│  │       │  srl_models.py           # 中层 对网络进行包装，按环境要求处理输入和输出
│  │       │  srl_network_builder.py  # 底层 构建神经网络 （actor+critic+disc）
│  │       └─ srl_players.py          # 用于演示
│  └─────tasks
│  │  └───  ...
│  │  └─────SRLEvo
│  │       │  humanoid_amp_srl_base.py  # 定义环境基类（与isaac gym的交互）
│  │       └─ humanoid_amp_srl.py  # 定义环境（观测、动作、奖励等）
└───assets 
   │  ...
   └─────mjcf
      └───  ...
      └───  amp_humanoid_srl.xml     # humanoid+外肢体模型文件 
      └───  amp_humanoid_srl_2.xml   # humanoid+外肢体模型文件（实验用）

```

## 4. 训练

**环境配置完成后，**在`IGEvolution\isaacgymenvs`下开启终端：

进行AMP实验：

```bash
python train.py task=HumanoidAMP experiment=AMP_run headless=True
```

使用训练好的模型查看结果：

```bash
python train.py task=HumanoidAMP num_envs=4 test=True  checkpoint='runs/SRL_walk_26-14-57-33.pth'
```

进行AMP+MARL（带外肢体）实验：

```bash
python SRL_Evo_train.py task=HumanoidAMPSRLTest experiment=SRL_walk task.env.asset.assetFileName='mjcf/amp_humanoid_srl_2.xml' headless=True max_iterations=2000  s
```

使用训练好的模型查看结果：

```bash
python SRL_Evo_train.py task=HumanoidAMPSRLTest  num_envs=4 test=True checkpoint='runs/SRL_walk_26-14-57-33.pth' task.env.asset.assetFileName='mjcf/amp_humanoid_srl_2.xml' 
```


