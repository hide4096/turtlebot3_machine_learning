# TurtleBot3
<img src="https://github.com/ROBOTIS-GIT/emanual/blob/master/assets/images/platform/turtlebot3/logo_turtlebot3.png" width="300">

## ROS Packages for TurtleBot3 Machine Learning

For Noetic + Ubuntu20.04

## Install

**Finish the simulation first.**
https://emanual.robotis.com/docs/en/platform/turtlebot3/simulation/#gazebo-simulation

``` bash
$ pip install msgpack argparse
$ pip install tensorflow

$ pip install keras tensorboard
$ pip install numpy pyqtgraph

$ cd ~/catkin_ws/src
$ git clone https://github.com/hide4096/turtlebot3_machine_learning.git
$ cd ~/catkin_ws
$ catkin_make
$ source ~/.bashrc

```

## Setup

### Set state

Modify the gazebo model file.
`turtlebot3_description/urdf/turtlebot3_burger.gazebo.xacro`

``` bash

<xacro:arg name="laser_visual" default="true"/>  # Modify default value to 'true'
```

``` bash

<scan>
  <horizontal>
    <samples>360</samples>            # The number of sample. Modify it to 24
    <resolution>1</resolution>
    <min_angle>0.0</min_angle>
    <max_angle>6.28319</max_angle>
  </horizontal>
</scan>
```

### Set parameters

DQN hyper parameter is here.
`turtlebot3_machine_learning/turtlebot3_dqn/nodes/turtlebot3_dqn_stage_#`

``` python

        self.load_model = False
        self.load_episode = 0
        self.state_size = state_size
        self.action_size = action_size
        self.episode_step = 6000
        self.target_update = 2000
        self.discount_factor = 0.99
        self.learning_rate = 0.00025
        self.epsilon = 1.0
        self.epsilon_decay = 0.99
        self.epsilon_min = 0.05
        self.batch_size = 8
        self.train_start = 8
        self.memory = deque(maxlen=1000000)
```

### Run

Modify '#' to stage number (1~4).
` $ roslaunch turtlebot3_gazebo turtlebot3_stage_#.launch `

Open another terminal and enter the command below.
` $ roslaunch turtlebot3_dqn turtlebot3_dqn_stage_#.launch `

You can see the visualized data.
` $ roslaunch turtlebot3_dqn result_graph.launch `

## Adjust

シミュレーター(Gazebo)と機械学習は時間同期してないので、マシンパワーが足りないとシミュレーションに学習が追いつかず、学習が進みません。
**Turtlebotが壁に当たっているのに元の位置に戻らない**時や**Episodeが進まない**時は学習が追いついていません

このような事態が発生した場合は次の手順を試してください

1. シミュレーション速度を下げる

`src/turtlebot3_simulations/turtlebot3_gazebo/worlds/turtlebot3_stage_#.world`

``` bash
<real_time_update_rate>1000.0</real_time_update_rate>
```
`1000.0`を半分にするとGazebo内の時間の進みも半分になります
`0`を指定すると、マシンパワーが許す限り高速で実行されます

`real_time_update_rate`は現実世界で1秒間に実行されるステップ数です
`max_step_size`はGazebo内での1ステップあたりの時間の増分(秒)です(いじらないほうがいい)
https://classic.gazebosim.org/tutorials?tut=physics_params&cat=physics

2. 負荷を下げる

DQN hyper parametersの`batch_size`と`train_size`を下げると負荷下がる気がする
よう分からんけど

設定を変えたら`catkin_make`を実行して反映させること