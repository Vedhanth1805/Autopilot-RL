# 🏎️ Autopilot-RL: Autonomous Self-Driving Car Simulation with Reinforcement Learning

[![Unity](https://img.shields.io/badge/Unity-6000.2.0f1%20(Unity%206)-blue.svg?logo=unity)](https://unity.com/)
[![ML-Agents](https://img.shields.io/badge/Unity%20ML--Agents-v4.0.2-orange.svg)](https://github.com/Unity-Technologies/ml-agents)
[![Algorithm](https://img.shields.io/badge/RL%20Algorithm-PPO-brightgreen.svg)]()
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

An end-to-end 3D autonomous driving simulation built with **Unity 6** and **Unity ML-Agents**. The autonomous vehicle learns optimal throttle, braking, and steering behaviors to navigate complex race tracks, avoid walls, and maintain high speeds using **Deep Reinforcement Learning (Proximal Policy Optimization - PPO)**.

---

## 📸 Highlights & Features

- **Physics-Accurate Vehicle Dynamics**: Driven by Unity `WheelCollider` physics with realistic suspension, motor torque, brake distribution, and dynamic center of mass offset.
- **7-Beam Lidar / Raycast Perception**: Custom raycast sensors scanning at $-60^\circ, -40^\circ, -20^\circ, 0^\circ, +20^\circ, +40^\circ, +60^\circ$ to perceive track boundaries and obstacles.
- **Dynamic Visual Lasers**: Real-time `LineRenderer` visual lasers that dynamically change color (white in open space, red when approaching barriers).
- **Real-Time Telemetry HUD**: In-game HUD displaying live speed ($m/s$), cumulative reward, step-by-step reward/penalty signals, episode countdown, and exact raycast distance readouts.
- **Pre-Trained Brains Ready for Inference**: Includes pre-trained ONNX models (`CarBehavior_v2.onnx`, `CarBehavior_v5-0.onnx`) allowing you to run the self-driving simulation out of the box with zero Python setup.
- **Versatile Operating Modes**:
  - **Inference Mode**: Runs the pre-trained neural network autonomously in Unity.
  - **Heuristic Mode**: Allows human player control using `WASD` or Arrow keys.
  - **Training Mode**: Connects live with Python ML-Agents PPO trainer for active learning.

---

## 🧠 Reinforcement Learning Formulation

### 1. State / Observation Space (11 Dimensions)
The agent observes the environment via an 11-dimensional continuous vector:

| Index | Observation | Normalization / Range | Description |
|:-----:|:------------|:----------------------|:------------|
| `0` | Normalized Forward Velocity | $v_{\text{forward}} / v_{\max}$ | Vehicle speed along forward axis |
| `1` | Normalized Lateral Velocity | $v_{\text{lateral}} / v_{\max}$ | Vehicle drift / slip velocity |
| `2` | Yaw Angular Velocity | $\omega_y / 5.0$ | Rate of vehicle rotation |
| `3` | Total Speed Magnitude | $\|\vec{v}\| / v_{\max}$ | Scalar velocity |
| `4 - 10` | 7 Raycast Distances | $d_i / d_{\text{ray}}$ | Normalized distances ($[-60^\circ \dots +60^\circ]$) |

### 2. Action Space (2 Continuous Actuators)
| Index | Action | Range | Mapping |
|:-----:|:-------|:------|:--------|
| `0` | Steering | $[-1.0, +1.0]$ | Steer angle up to $\pm 30^\circ$ on front wheels |
| `1` | Throttle / Brake | $[-1.0, +1.0]$ | $> 0$: Forward motor torque<br>$< 0$: Front & rear braking torque |

### 3. Reward Function
The dense reward function incentivizes safe, smooth, high-speed track navigation:

$$\mathcal{R}_t = \underbrace{0.005 \cdot v_{\text{forward}}}_{\text{Speed Reward}} - \underbrace{0.001 \cdot |a_{\text{steer}}|}_{\text{Smooth Steering Penalty}} - \underbrace{\mathbb{I}_{v < 0.5}(0.005)}_{\text{Idling Penalty}} - \underbrace{\mathbb{I}_{v < -1.0}(0.01)}_{\text{Reversing Penalty}}$$

- **Wall Collision**: Terminal penalty of **$-2.0$** and immediate episode reset upon impact with track barriers.
- **Episode Timeout**: Max episode time of 90 seconds.

---

## 📁 Repository Structure

```text
Autopilot-RL/
├── Assets/
│   ├── CarBehavior_v2.onnx        # Pre-trained neural network model
│   ├── CarBehavior_v5-0.onnx      # Checkpointed model
│   ├── RL/
│   │   ├── Scenes/
│   │   │   ├── HomeScreen.unity   # Main menu scene
│   │   │   ├── RL_Scene.unity     # Primary reinforcement learning track
│   │   │   └── GameScene.unity    # Extended track scene
│   │   └── Scripts/
│   │       ├── CarMLAgent.cs      # Core agent script (Observations, Actions, Rewards)
│   │       ├── CubeMLAgent.cs     # Simple benchmark agent
│   │       ├── CarFollowCamera.cs # Smooth follow camera
│   │       └── PlayButtonController.cs
│   └── Racing Starter Kit/        # Vehicle models, tracks, audio, shaders
├── Packages/
│   └── manifest.json              # Package dependencies (com.unity.ml-agents 4.0.2)
├── ProjectSettings/
│   ├── ProjectVersion.txt         # Unity 6000.2.0f1
│   └── EditorBuildSettings.asset
├── config/
│   ├── car_training.yaml          # PPO training configuration (v2)
│   ├── car_config.yaml            # PPO training configuration (v5)
│   └── cube_training.yaml
├── results/                       # Training logs, checkpoints, TensorBoard summaries
└── README.md
```

---

## 🚀 Quick Start: Running in Unity

### Requirements
- **[Unity Hub](https://unity.com/download)**
- **Unity Editor 6000.2.0f1** (Unity 6)

### Steps
1. **Clone the Repository**:
   ```bash
   git clone https://github.com/Vedhanth1805/Autopilot-RL.git
   cd Autopilot-RL
   ```
2. **Open in Unity Hub**:
   - Launch **Unity Hub**.
   - Click **Add** -> **Add project from disk**.
   - Select the cloned `Autopilot-RL` directory.
   - Open using Unity **6000.2.0f1**.
3. **Run the Autonomous Agent**:
   - In the Unity Project browser, navigate to:
     `Assets` → `RL` → `Scenes` → double-click **`RL_Scene.unity`**.
   - Click the **Play (▶)** button at the top of the editor.
   - The autonomous vehicle will immediately begin navigating the track using the pre-loaded ONNX model!

---

## 🏋️ Training with ML-Agents (Optional)

If you wish to train the neural network from scratch or fine-tune existing models:

### 1. Setup Python Environment
We recommend Python 3.9 or 3.10:
```bash
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install mlagents torch
```

### 2. Start Training
Run the training command from the project root:
```bash
mlagents-learn config/car_training.yaml --run-id=CarRun_New --time-scale=20
```

### 3. Connect Unity
1. In Unity, select the **Car** GameObject in the hierarchy.
2. In the `Behavior Parameters` component, set **Behavior Type** to `Default`.
3. Press **Play (▶)** in the Unity Editor when prompted in the terminal.
4. Monitor training performance live with TensorBoard:
   ```bash
   tensorboard --logdir results
   ```

---

## ⚙️ Hyperparameter Configuration (`car_training.yaml`)

```yaml
behaviors:
  CarBehavior_v2:
    trainer_type: ppo
    hyperparameters:
      batch_size: 512
      buffer_size: 4096
      learning_rate: 0.0003
      learning_rate_schedule: linear
      beta: 0.005
      epsilon: 0.2
      lambd: 0.95
      num_epoch: 5
    network_settings:
      normalize: true
      hidden_units: 256
      num_layers: 3
    reward_signals:
      extrinsic:
        gamma: 0.99
        strength: 1.0
    max_steps: 1000000
    time_horizon: 128
    summary_freq: 5000
```

---

## 🎮 Manual Driving (Heuristic Mode)

To test the track physics or drive the vehicle manually:
1. In the Unity Hierarchy, select the **Car** GameObject.
2. In the `Behavior Parameters` component, change **Behavior Type** to **`Heuristic Only`**.
3. Press **Play (▶)** and use:
   - `W` / `Up Arrow`: Accelerate
   - `S` / `Down Arrow`: Brake / Reverse
   - `A` / `Left Arrow`: Steer Left
   - `D` / `Right Arrow`: Steer Right

---

## 👤 Author & Acknowledgments

- **Repository**: [https://github.com/Vedhanth1805/Autopilot-RL](https://github.com/Vedhanth1805/Autopilot-RL)
- **Author**: [Vedhanth Dharbha](https://github.com/Vedhanth1805)
- **Framework**: [Unity ML-Agents Toolkit](https://unity-technologies.github.io/ml-agents/)
