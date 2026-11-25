

<div style="background-color: #fff8eb; border: 1px solid #f1d8a6; border-radius: 6px; padding: 14px 18px; margin: 16px 0; line-height: 1.7;">
  <p style="margin: 0 0 8px;">❗ <strong>System Requirements</strong>:</p>
  <div style="margin-left: 28px;">
    <p id="minimum-training-configuration" style="margin: 8px 0; scroll-margin-top: 100px;"><strong>Minimum Training Configuration</strong>:</p>
    <ul style="margin: 0; padding-left: 20px;">
      <li><strong>Python</strong>: 3.11 or later (the <code>uv</code> package manager is recommended)</li>
      <li><strong>GPU</strong>: NVIDIA GPU required; A100 or H100 recommended</li>
      <li><strong>Memory</strong>: 32 GB or more of system RAM</li>
      <li><strong>Storage</strong>: 100 GB or more of available space on an NVMe SSD</li>
      <li><strong>Operating system</strong>: Ubuntu 22.04 LTS or later</li>
      <li><strong>Network</strong>: Access to GitHub</li>
    </ul>
    <p id="minimum-deployment-configuration" style="margin: 12px 0 8px; scroll-margin-top: 100px;"><strong>Minimum Deployment Configuration</strong>:</p>
    <ul style="margin: 0; padding-left: 20px;">
      <li><strong>Python</strong>: 3.11 or later (the <code>uv</code> package manager is recommended)</li>
      <li><strong>GPU</strong>: NVIDIA GPU with at least 12 GB of VRAM; GeForce RTX 3090 (24 GB) or better recommended</li>
      <li><strong>CPU</strong>: At least six cores; Intel Core i7-10700K, AMD Ryzen 5 5600X, or better recommended</li>
      <li><strong>Memory</strong>: 16 GB or more of system RAM</li>
      <li><strong>Storage</strong>: At least 30 GB of available space</li>
      <li><strong>Operating system</strong>: Ubuntu 20.04 LTS or later</li>
      <li><strong>Network</strong>: Access to GitHub</li>
    </ul>
    <p style="margin: 12px 0 8px;"><strong>Material Purchase Links</strong>:</p>
    <ul style="margin: 0; padding-left: 20px;">
      <li><a style="color: #4664ff;" href="https://item.jd.com/100141805181.html?pcdk=0idlyuv_3YF5YjpmwH6Die7WiZBZN0X7hHC7KFRJ6dlDMPl4CCz2MK5jYRcLMWXk.3z6a.aI3x&amp;spmTag=YTAyNDAuYjAwMjQ5My5jMDAwMDQwMjcuMSUyM3NrdV9jYXJkJTQwMTc4NDYwNTIzOTgxNCUyMzE3ODQ2MDQ3MTkxOTExODg4OTM4NTk0JTIzMTg5MDQ0MDY3MA"><strong>Candy</strong></a>: One box of "Big White Rabbit 500g Money Rabbit Like Gift Box"</li>
      <li><a style="color: #4664ff;" href="https://item.jd.com/10175580708819.html?pcdk=rmQphIP94gstOLhv682n5e5Gs9BVITvLf6co0I0LISSAxlr-PeQoKdyOZWSdhNZa.3z6a.aI3x&amp;spmTag=YTAyNDAuYjAwMjQ5My5jMDAwMDQwMjcuMSUyM3NrdV9jYXJkJTQwMTc4NDYwNTM5OTkxNCUyMzE3ODQ2MDQ3MTkxOTExODg4OTM4NTk0JTIzMTY1Mzc3MTQ1OQ"><strong>Cream-colored fruit plate</strong></a>: Use the "Luxury Fruit Plate 2-in-1 [Cream White + Transparent]" in cream color for the fruit plate</li>
    </ul>
  </div>
</div>

## <span style="color: #4664ff;">1.</span> Task Overview

This release provides a complete solution for the candy dispensing task implemented based on Pi05, including the open-sourced dataset, model weights, and training and inference source code. The task triggers candy dispensing upon recognition of a human hand gesture—specifically, a palm-up orientation—and is designed to validate TRON2's fundamental pick-and-place atomic operation capabilities in target perception and end-effector tracking scenarios. Developers may refer to the standardized development pipeline presented in this project to migrate the complete technical framework to various pick-and-place robotic tasks. Leveraging the accompanying data acquisition platform and embodied robotics platform, users can independently carry out the end-to-end development of custom tasks and rapidly build embodied-AI robotic application scenarios tailored to their specific business requirements.

The development workflow for the dual-arm manipulation project comprises three main stages: [Data Collection](#2-data-collection) → [Model Training](#4-model-training) → [Model Deployment](#5-model-deployment).

<div style="background-color: #fff8eb; border: 1px solid #f1d8a6; border-radius: 6px; padding: 14px 18px; margin: 16px 0; line-height: 1.7;">
  <p style="margin: 0 0 8px;">💡 <strong>Training Is Optional</strong>:</p>
  <p style="margin: 0 0 0 28px;">You do not need to train a model to try the candy dispensing task. To evaluate the result quickly, skip Sections <a style="color: #4664ff;" href="#2-data-collection">2</a>–<a style="color: #4664ff;" href="#4-model-training">4</a>, download the released pre-trained weights (<a style="color: #4664ff;" href="#512-weight-download">Section 5.1.2</a>), and go straight to deployment (<a style="color: #4664ff;" href="#5-model-deployment">Section 5</a>). Collect data and train a model yourself only if you want to customize the task or reproduce the training results.</p>
</div>

Pi0.5 is a Vision-Language-Action (VLA) model developed by Physical Intelligence. For those intending to fine-tune such models using data exported from embodied-data platforms, this guide provides a walkthrough of the complete pipeline based on the official OpenPI framework. It also describes how to perform efficient fine-tuning of the pre-trained Pi0.5 model on custom datasets.

## <span id="2-data-collection" style="color: #4664ff; scroll-margin-top: 100px;">2.</span> Data Collection

### <span style="color: #4664ff;">2.1</span> Environment Setup

For the candy dispensing task, in order to better leverage the datasets that have been collected and publicly released by LimX Dynamics, it is recommended to replicate the official data collection environment as closely as possible. The specific setup requirements are as follows:

<img src="https://oss.limxdynamics.com/uploads/material/%E9%BB%98%E8%AE%A4/TRON%202/demo/candy/Candy_Dispensing_TRON2_Dual-Ar_0_sec_en.png" alt="Candy dispensing workspace setup" width="250" align="right" loading="lazy" style="float: right; width: 50%; max-width: 500px; height: auto; margin: 0 0 12px 20px; border-radius: 4px;">

- **Robot Mode**: The robot is switched to teleoperation mode via the controller (TeleOperator).
- **Table Specifications**: Dimensions of 160 × 80 × 75 cm (length × width × height), with a pure white tabletop. (The height should be kept as consistent as possible.)
- **Robot Placement**: With the fixing bracket in place, the robot is positioned such that the edge of the robot is 15 cm from the edge of the table.
- **Prop Placement**: Candies are densely placed in the central area of the tabletop near the robot side, with a beige serving tray placed underneath. The placement should ensure that both arms of the robot are able to reach and grasp the candies.

<div style="clear: both;"></div>

### <span style="color: #4664ff;">2.2</span> Connecting the TRON2 Robot

Connect your computer to the robot by following [Section 6: Robot Network Connection](https://www.limxdynamics.com/en/documents/852949782845591552#6-Robot-Network-Connection) in the *TRON 2 User Manual*.

### <span style="color: #4664ff;">2.3</span> Accessing the Data Acquisition Platform

Make sure the external development expansion module is connected to the robot by Ethernet and your computer is connected to the robot's Wi-Fi. Alternatively, connect your computer directly to an Ethernet port on the module. Open [https://10.192.1.4/](https://10.192.1.4/) in your browser to access the local data acquisition interface.



### <span style="color: #4664ff;">2.4</span> TRON 2 Robot Configuration

<img src="https://oss.limxdynamics.com/uploads/material/%E9%BB%98%E8%AE%A4/TRON%202/demo/candy/Candy_Dispensing_TRON2_Dual-Ar_0_sec_2_en.png" alt="TRON 2 teleoperation preparation" width="300" align="right" loading="lazy" style="float: right; width: 50%; max-width: 400px; height: auto; margin: 0 0 12px 20px; border-radius: 4px;">

- **Robot Status**: After switching the controller to TeleOperator mode, the robot's head will automatically adjust to its lowest posture and rotate to the center position (`pitch = 1.04`, `yaw = 0`). Once connected to the robot, check whether the viewing angle is appropriate on the data acquisition software overview page at [https://10.192.1.4/](https://10.192.1.4/).
- Follow [Section 5.3: Teleoperation Preparation](https://www.limxdynamics.com/en/documents/852949782845591552#5.3-Teleoperation-Preparation) and [Section 5.4: Using Teleoperation](https://www.limxdynamics.com/en/documents/852949782845591552#5.4-Using-Teleoperation) in the *TRON 2 User Manual* to prepare the robot for teleoperation.
- If a robot body firmware upgrade changes the teleoperation button mapping, update the data collection software on the Developer Expansion Module and restart the robot. Follow [Section 7.4: Updating the Data Collection Software](https://www.limxdynamics.com/en/documents/852949782845591552#7.4-Updating-the-Data-Collection-Software-Developer-Expansion-Module) in the *TRON 2 User Manual*.

<div style="clear: both;"></div>

### <span style="color: #4664ff;">2.5</span> Data Collection Settings

<div style="background-color: #fff8eb; border: 1px solid #f1d8a6; border-radius: 6px; padding: 14px 18px; margin: 16px 0; line-height: 1.7;">
  <p style="margin: 0 0 8px;">⚠️ <strong>Before you start collecting data, also check the following:</strong></p>
  <ul style="margin: 0; padding-left: 40px;">
    <li>The network connection is working.</li>
    <li>The data disk has enough free space (recording a 1-minute episode takes about 800 MB–1.5 GB).</li>
  </ul>
</div>

Follow the Data Recording section on the Overview page in the [LimX Dynamics Data Collection Software User Manual](https://www.limxdynamics.com/en/documents/903336680742850560), and set the parameters in the **data collection software**.

<table style="border-collapse: collapse; width: 100%; table-layout: fixed;">
  <colgroup>
    <col style="width: 110px;">
    <col>
  </colgroup>
  <thead>
    <tr style="background-color: #f5f7fa;">
      <th scope="col" style="border: 1px solid #e0e0e0; padding: 6px 10px; text-align: center; vertical-align: middle;"><strong>Parameter</strong></th>
      <th scope="col" style="border: 1px solid #e0e0e0; padding: 6px 10px; text-align: left; vertical-align: middle; word-wrap: break-word;"><strong>Setting</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="border: 1px solid #e0e0e0; padding: 6px 10px; text-align: center; vertical-align: middle;"><strong>File Name</strong></td>
      <td style="border: 1px solid #e0e0e0; padding: 6px 10px; text-align: left; vertical-align: middle; word-wrap: break-word;">Enter any name you like.</td>
    </tr>
    <tr>
      <td style="border: 1px solid #e0e0e0; padding: 6px 10px; text-align: center; vertical-align: middle;"><strong>Domain ID</strong></td>
      <td style="border: 1px solid #e0e0e0; padding: 6px 10px; text-align: left; vertical-align: middle; word-wrap: break-word;">Enter 0 in most cases. If multiple robots share the same network, assign a separate ID to each robot as needed.</td>
    </tr>
    <tr>
      <td style="border: 1px solid #e0e0e0; padding: 6px 10px; text-align: center; vertical-align: middle;"><strong>Format</strong></td>
      <td style="border: 1px solid #e0e0e0; padding: 6px 10px; text-align: left; vertical-align: middle; word-wrap: break-word;">mcap</td>
    </tr>
    <tr>
      <td style="border: 1px solid #e0e0e0; padding: 6px 10px; text-align: center; vertical-align: middle;"><strong>Topic</strong></td>
      <td style="border: 1px solid #e0e0e0; padding: 6px 10px; text-align: left; vertical-align: top; word-wrap: break-word;">Select the following 13 topics (recommended):<pre style="background-color: #f6f8fa; color: #24292f; border-radius: 6px; padding: 10px 14px; margin: 8px 0; overflow-x: auto; font-family: monospace; text-align: left; white-space: pre-wrap; word-break: break-word;"><code>1. /camera/left/color/image_resized/compressed<br>   sensor_msgs/msg/CompressedImage<br>2. /camera/right/color/image_resized/compressed<br>   sensor_msgs/msg/CompressedImage<br>3. /camera/top/color/image_raw/compressed<br>   sensor_msgs/msg/CompressedImage<br>4. /gripper_command<br>   sensor_msgs/msg/JointState<br>5. /gripper_state<br>   sensor_msgs/msg/JointState<br>6. /joint_command<br>   sensor_msgs/msg/JointState<br>7. /joint_states<br>   sensor_msgs/msg/JointState<br>8. /left_arm/ee_pose<br>   geometry_msgs/msg/PoseStamped<br>9. /left_arm/ee_pose_cmd<br>   geometry_msgs/msg/PoseStamped<br>10. /right_arm/ee_pose<br>    geometry_msgs/msg/PoseStamped<br>11. /right_arm/ee_pose_cmd<br>    geometry_msgs/msg/PoseStamped<br>12. /lifter/state<br>13. /chassis/twist</code></pre></td>
    </tr>
  </tbody>
</table>

**Data collection procedure:**

1. **Move both arms to the ready pose**: Press and hold the Grip Button + X on the left VR controller and the Grip Button + A on the right VR controller at the same time. After about 0.5 s, both arms start moving to the ready pose. Release the buttons to stop the motion immediately.

2. **Start recording**: Press Y on the left VR controller (or click the green start icon under **Overview** → **Data Recording** in the data collection software).

3. **Perform the collection task**: Teleoperate the robot to pick up a candy from the table and hand it to the person. Follow this rule:

   - The arms start moving to pick up and hand over the candy only after the system recognizes a palm-up hand. While no palm-up hand is detected, keep the arms still.

4. **Reset**: Press L1 + X on the Remote Controller, or press the Grip Button + B on the right VR controller, to return both arms.

5. **End recording**: Press B on the right VR controller (or click the end icon under **Overview** → **Data Recording** in the data collection software).

6. Repeat steps 1–5 to record more episodes.

<div style="background-color: #fff8eb; border: 1px solid #f1d8a6; border-radius: 6px; padding: 14px 18px; margin: 16px 0; line-height: 1.7;">
  <p style="margin: 0;">💡 Additionally, we have open-sourced a data conversion script: <a style="color: #4664ff;" href="https://oss.limxdynamics.com/uploads/attachment/2026/09/23/1790164596233_limx_mcap2lerobot.zip">📎 limx_mcap2lerobot.zip</a>.</p>
</div>

### <span style="color: #4664ff;">2.6</span> Dataset Validation

```text
~/Desktop/lerobot_dataset$ tree -d
.
├── data
│   └── chunk-000
├── meta
│   └── episodes
│       └── chunk-000
└── videos
    ├── observation.images.cam_high
    │   └── chunk-000
    ├── observation.images.cam_left_wrist
    │   └── chunk-000
    └── observation.images.cam_right_wrist
        └── chunk-000
```

## <span style="color: #4664ff;">3.</span> Dataset Description

- Dataset Statistics:

<table style="border-collapse: collapse; width: 100%; color: #303846;">
  <thead>
    <tr style="background-color: #f5f7fa;">
      <th scope="col" style="border: 1px solid #e0e0e0; padding: 8px 12px; text-align: center;">Metric</th>
      <th scope="col" style="border: 1px solid #e0e0e0; padding: 8px 12px; text-align: center;">Value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; text-align: center;">Data Description</td>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; text-align: center;">1953 episodes</td>
    </tr>
    <tr>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; text-align: center;">Data URL</td>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; text-align: center; overflow-wrap: anywhere;"><a style="color: #4664ff;" href="https://huggingface.co/limx-tron2/tron2-openpi-models">https://huggingface.co/limx-tron2/tron2-openpi-models</a></td>
    </tr>
    <tr>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; text-align: center;">Task Type</td>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; text-align: center;">Candy Dispensing (Pick &amp; Place)</td>
    </tr>
    <tr>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; text-align: center;">Collection Method</td>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; text-align: center;">Pico (VR Teleoperation)</td>
    </tr>
    <tr>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; text-align: center;">Data Format</td>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; text-align: center;">Lerobot2.1</td>
    </tr>
  </tbody>
</table>

## <span id="4-model-training" style="color: #4664ff; scroll-margin-top: 100px;">4.</span> Model Training

<div style="display: flex; align-items: flex-start; gap: 8px; background-color: #fff8eb; border: 1px solid #f1d8a6; border-radius: 6px; padding: 14px 18px; margin: 16px 0; line-height: 1.7;">
  <span style="flex-shrink: 0; font-size: 16px;">💡</span>
  <div><strong>Skip this section if you use the released pre-trained weights.</strong> Go directly to <a style="color: #4664ff;" href="#5-model-deployment">Section 5 Model Deployment</a>.</div>
</div>

<table style="border-collapse: collapse; width: 100%; color: #303846;">
  <thead>
    <tr style="background-color: #f5f7fa;">
      <th scope="col" style="border: 1px solid #e0e0e0; padding: 8px 12px; text-align: center;">Training Method</th>
      <th scope="col" style="border: 1px solid #e0e0e0; padding: 8px 12px; text-align: center;">Use Case</th>
      <th scope="col" style="border: 1px solid #e0e0e0; padding: 8px 12px; text-align: center;">Advantages</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; vertical-align: top;">Training with Self-Hosted Compute</td>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; vertical-align: top;">Experienced developers with existing compute resources (local servers or cloud resources) who are familiar with cloud development</td>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; vertical-align: top;"><ul style="margin: 0; padding-left: 18px;"><li>Reuse your existing compute resources</li><li>Open-source training and inference source code for greater openness and iterative optimization <strong>(non-commercial)</strong></li></ul></td>
    </tr>
  </tbody>
</table>

### <span style="color: #4664ff;">4.1</span> Training with Self-Hosted Compute

<div style="background-color: #fff8eb; border: 1px solid #f1d8a6; border-radius: 6px; padding: 14px 18px; margin: 16px 0; line-height: 1.7;">
  <p style="margin: 0 0 8px;">⚠️ <strong>Note</strong>:</p>
  <p style="margin: 0 0 0 28px;">Before training models with self-hosted compute, please refer to <a style="color: #4664ff;" href="#minimum-training-configuration">Minimum Training Configuration</a> at the beginning of this document.</p>
</div>

<br/>

<div style="background-color: #fff8eb; border: 1px solid #f1d8a6; border-radius: 6px; padding: 14px 18px; margin: 16px 0; line-height: 1.7;">
  <p style="margin: 0 0 8px;">💡 <strong>About the Training Framework</strong>:</p>
  <ul style="margin: 0 0 0 28px; padding-left: 20px;">
    <li>The underlying technology stack of the OpenPI open-source project includes both JAX and PyTorch frameworks for training its core model Pi05. Since OpenPI's native training framework is JAX, and the core model, training pipeline, and base checkpoints are all built on JAX/Flax, this project defaults to the JAX framework for training.</li>
    <li>JAX is a computational tool for model training, similar in positioning to PyTorch—it is not a model or an algorithm, but simply "a framework for running models."</li>
    <li>JAX's advantage lies in its high computational efficiency, making it particularly suitable for large model training scenarios. This project selects JAX precisely because it is more efficient for training large models like Pi0.5.</li>
  </ul>
</div>

If you wish to train models with open-source or custom datasets in a self-hosted compute environment, you may refer to the procedure below in conjunction with the open-source code. The current open-source code has completed the adaptation of the OpenPI source code to TRON2. This document uses Pi05 as an example for illustration.

#### <span style="color: #4664ff;">4.1.1</span> Environment Setup

```bash
git clone https://github.com/limx-tron2/tron2_openpi.git
git clone https://github.com/limx-tron2/tron2_env.git

cd tron2_openpi

sudo apt install pkg-config
sudo apt install software-properties-common -y
sudo add-apt-repository ppa:ubuntuhandbook1/ffmpeg7 -y
sudo apt install ffmpeg libavformat-dev libavcodec-dev libavdevice-dev libavutil-dev libswscale-dev libswresample-dev libavfilter-dev -y

uv sync
uv pip install -e .

source .venv/bin/activate
uv pip install -e "../tron2_env[bridge,openpi]" # Activate the current environment

# Verify successful installation; should output the path: ../tron2_env/src/tron2_env/__init__.py
PYTHONPATH="$(cd ../tron2_env/src && pwd)" uv run python -c "import tron2_env; print(tron2_env.__file__)"
```

#### <span style="color: #4664ff;">4.1.2</span> Training Procedure

1. It is recommended to manage parameters using independent YAML task configuration files rather than directly modifying `src/openpi/training/config.py`. This facilitates configuration isolation, version management, and reusability:

   ```bash
   cp configs/train/tron2_tasks/example.yaml configs/train/tron2_tasks/my_task.yaml
   ```

2. Open the configuration file `configs/train/tron2_tasks/my_task.yaml` and modify the dataset path parameter `repo_id: my_dataset`. The system constructs the full dataset path via the environment variable `HF_LEROBOT_HOME`. The local directory must follow a fixed structure:

   - `$HF_LEROBOT_HOME/my_dataset/data/`: stores raw data such as images, trajectories, and actions.
   - `$HF_LEROBOT_HOME/my_dataset/meta/`: stores dataset statistics, task descriptions, and episode information.

   ```bash
   export HF_LEROBOT_HOME=/path/to/datasets
   ```

3. Dataset normalization parameters are a necessary prerequisite for training convergence. Before the first training run, execute the normalization statistics script (`compute_norm_stats.py`) to compute the statistics file:

   ```bash
   uv run scripts/compute_norm_stats.py \
     --task-config configs/train/tron2_tasks/my_task.yaml
   ```

4. After the statistics computation is complete, execute the training script to start training:

   ```bash
   uv run scripts/train_tron2_task.py \
     --task-config configs/train/tron2_tasks/my_task.yaml
   ```

<div style="background-color: #fff8eb; border: 1px solid #f1d8a6; border-radius: 6px; padding: 14px 18px; margin: 16px 0; line-height: 1.7;">
  <p style="margin: 0 0 8px;">❗ <strong>Note</strong>:</p>
  <p style="margin: 0 0 8px 28px;">We provide an all-in-one entry script, <code>scripts/cloud_train_entrypoint_portable.sh</code>, that eliminates the need for step-by-step execution:</p>
  <ul style="margin: 0 0 0 28px; padding-left: 20px;">
    <li>When the <code>--skip-norm</code> parameter is not provided, the script will automatically compute normalization statistics first, then start training.</li>
    <li>To skip the statistics computation, add the <code>--skip-norm</code> parameter.</li>
  </ul>
</div>

##### Local Custom Path Mode

When self-hosted compute does not have a unified mount directory, manually specify the full paths for the dataset, weights, and output:

```bash
scripts/cloud_train_entrypoint_portable.sh \
  --data-dir /path/to/datasets \
  --repo-id my_dataset \
  --weight /path/to/checkpoint/params \
  --output-dir /path/to/output \
  --exp my_task \
  --prompt "perform the configured task" \
  --max-frames 100000
```

##### Custom YAML Configuration File Mode

Directly load the pre-edited `my_task.yaml`, which centrally manages parameters such as `repo_id`, `weight_loader`, `assets_base_dir`, and `checkpoint_base_dir`. You only need to specify `HF_LEROBOT_HOME` via `--data-dir`:

```bash
scripts/cloud_train_entrypoint_portable.sh \
  --task-config configs/train/tron2_tasks/my_task.yaml \
  --exp my_task \
  --data-dir "$HF_LEROBOT_HOME" \
  --max-frames 100000
```

<div style="background-color: #fff8eb; border: 1px solid #f1d8a6; border-radius: 6px; padding: 14px 18px; margin: 16px 0; line-height: 1.7;">
  <p style="margin: 0 0 8px;">❗ <strong>Additional Notes</strong>:</p>
  <ol style="margin: 0 0 0 28px; padding-left: 20px;">
    <li>Custom task configuration files (e.g. <code>my_task.yaml</code>) are already configured in <code>.gitignore</code> and will not be committed to the code repository; the repository only retains the generic model config <code>configs/train/tron2_tasks/example.yaml</code>.</li>
    <li>The template YAML supports the following configuration items: dataset <code>repo_id</code>, task text <code>prompt</code>, data column definitions, action prediction horizon <code>action_horizon</code>, state dimension <code>state_dim</code>, pre-trained base checkpoint, training output path, and optional <code>prompt_from_task</code> and <code>rtc_training_simulated_delay</code>.</li>
  </ol>
</div>

## <span id="5-model-deployment" style="color: #4664ff; scroll-margin-top: 100px;">5.</span> Model Deployment

You can deploy either the released pre-trained weights (no training required) or weights you trained yourself in [Section 4](#4-model-training). To use the released weights, follow Sections [5.1.1](#511-environment-setup) to [5.1.5](#515-start-policy-server) as written.

### <span style="color: #4664ff;">5.1</span> Inference Deployment Procedure

<div style="background-color: #fff8eb; border: 1px solid #f1d8a6; border-radius: 6px; padding: 14px 18px; margin: 16px 0; line-height: 1.7;">
  <p style="margin: 0 0 8px;">⚠️</p>
  <ol style="margin: 0; padding-left: 28px;">
    <li><strong>Robot initial posture requirement</strong>: Ensure the robot is in the teleoperation initial state (press L1+X on the Remote Controller, or press the Grip Button + B on the right VR controller, to return both arms), with both arms naturally vertical against the sides of the body. After starting the client, the robot will automatically extend both arms sideways and then forward. <strong>If not in the initial state</strong>, the robot may directly raise both arms. <strong>Be cautious</strong> of obstacles in front.</li>
    <li>Before inference, the robot controller must be switched to <strong>High-Level Developer Mode (High-Level Dev)</strong>.
      <ul style="padding-left: 20px;"><li>If the table height, bracket distance from the table edge, lighting environment, and head angle were not changed during data collection, they must remain consistent during inference.</li></ul>
    </li>
    <li>For other environment-related constraints, please refer to <a style="color: #4664ff;" href="#minimum-deployment-configuration"><strong>Minimum Deployment Configuration</strong></a> at the beginning of this document.</li>
    <li>Camera connection mode description:
      <ol type="a" style="padding-left: 20px;">
        <li><strong>Bridge Mode</strong>: Pull image data via TRON2 Bridge, and simultaneously use robot state information calibrated by Bridge.</li>
        <li><strong>Legacy Local Direct Connection Mode</strong>: The local inference host directly connects to RealSense cameras and obtains state from the robot's WebSocket (in this case, the USB ports of the robot's three cameras must be connected to the inference chassis).</li>
      </ol>
    </li>
    <li>When <code>client.max_steps</code> is <code>null</code>, the client needs to be stopped manually.</li>
  </ol>
</div>

#### <span id="511-environment-setup" style="color: #4664ff; scroll-margin-top: 100px;">5.1.1</span> Environment Setup

```bash
git clone https://github.com/limx-tron2/tron2_openpi.git
git clone https://github.com/limx-tron2/tron2_env.git

cd tron2_openpi

sudo apt install pkg-config
sudo apt install software-properties-common -y
sudo add-apt-repository ppa:ubuntuhandbook1/ffmpeg7 -y
sudo apt install ffmpeg libavformat-dev libavcodec-dev libavdevice-dev libavutil-dev libswscale-dev libswresample-dev libavfilter-dev -y

uv sync
uv pip install -e .

source .venv/bin/activate
uv pip install -e "../tron2_env[bridge,openpi]"

# Verify successful installation; should output the path: ../tron2_env/src/tron2_env/__init__.py
PYTHONPATH="$(cd ../tron2_env/src && pwd)" uv run python -c "import tron2_env; print(tron2_env.__file__)"
```

#### <span id="512-weight-download" style="color: #4664ff; scroll-margin-top: 100px;">5.1.2</span> Weight Download

The `pi05_tron2_candy` weights are already trained on the candy dispensing dataset. You can use them directly without training. Download them into the `checkpoints` folder under `/home/linux_user_name/xxx/tron2_openpi` from [Hugging Face](https://huggingface.co/limxdynamics/tron2-openpi-models/tree/main).

```bash
cd /home/xxx/xxx/tron2_openpi
mkdir checkpoints
hf download limxdynamics/tron2-openpi-models \
  --repo-type model \
  --include "pi05_tron2_candy/**" \
  --local-dir ./checkpoints
```

After the download completes, the checkpoint is located at `checkpoints/pi05_tron2_candy/candy_h30_rtc_25000`.

#### <span style="color: #4664ff;">5.1.3</span> Deployment Configuration

1. The three YAML files in `/home/linux_user_name/xxx/tron2_openpi/configs/deploy` are our example configurations. Among them, `candy_server.yaml` and `candy_client.yaml` are public example configurations for the Candy task, containing various parameters that you can modify according to the comments.
2. Before using with your own robot, please **carefully verify** that the checkpoint path, robot address, Bridge address, initial posture, and safety procedures are all appropriate for your deployment.
3. Common parameters to modify (minimum local configuration example shown below; the address refers to the policy content in any `candy_server.yaml` file you use under `tron2_openpi/configs/deploy/`; detailed configuration descriptions are written in the YAML comments):

```yaml
policy:
  config: TASK_CONFIG_NAME # Training configuration name; corresponds to lines 642-653 of /home/xxx/xxx/tron2_openpi/src/openpi/training/config.py
  repo_id: DATASET_REPO_ID # Must match the asset subdirectory of the checkpoint file; our open-source weights default to "input"
  checkpoint_dir: /path/to/checkpoints/TASK_CONFIG_NAME/experiment/step # Checkpoint path; can be an absolute path or a relative path to tron2_openpi
  default_prompt: TASK_PROMPT # Candy dispensing demo defaults to "Hand out the candy"
robot:
  ip: ROBOT_IP
client:
  observation_source: bridge
  max_steps: null
```

If you use the released pre-trained weights, `configs/deploy/candy_server.yaml` already contains the matching values. You only need to confirm the following:

```yaml
policy:
  config: pi05_tron2_candy
  repo_id: input
  checkpoint_dir: checkpoints/pi05_tron2_candy/candy_h30_rtc_25000
  default_prompt: Hand out the candy
robot:
  ip: ROBOT_IP # Replace with your robot's IP address
```

If you trained the model yourself, set `config`, `repo_id`, and `checkpoint_dir` to match your training output.

#### <span style="color: #4664ff;">5.1.4</span> Robot Environment

1. **Ensure** the robot is in the teleoperation initial state (press L1+X on the Remote Controller, or press the Grip Button + B on the right VR controller, to return both arms), i.e., both arms are vertically positioned at the sides.
2. Switch the controller to High-Level Developer Mode (High-Level-Develop).

#### <span id="515-start-policy-server" style="color: #4664ff; scroll-margin-top: 100px;">5.1.5</span> Start Policy Server

1. Start the policy server in one terminal:

   ```bash
   cd tron2_openpi
   uv run scripts/serve_policy.py   --profile configs/deploy/candy_server.yaml
   ```

2. Start the TRON2 client in another terminal:

   ```bash
   cd tron2_openpi
   uv run examples/tron2/pi_client_rtc.py   --profile configs/deploy/candy_client.yaml
   ```

3. After running the client, the robotic arms will smoothly transition from the vertical side position to the forearm-vertical side position. Subsequently, you can place your palm-up hand within the camera's field of view for inference verification.

## <span style="color: #4664ff;">6.</span> Code Repositories and Resources

<table style="border-collapse: collapse; width: 100%; color: #303846;">
  <thead>
    <tr style="background-color: #f5f7fa;">
      <th scope="col" style="border: 1px solid #e0e0e0; padding: 8px 12px; text-align: center;">Resource</th>
      <th scope="col" style="border: 1px solid #e0e0e0; padding: 8px 12px; text-align: center;">Description</th>
      <th scope="col" style="border: 1px solid #e0e0e0; padding: 8px 12px; text-align: center;">Link</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; vertical-align: top;">tron2_openpi</td>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; vertical-align: top;">Training and inference source code (OpenPI adapted for TRON2), including the deployment configurations in <code>configs/deploy</code></td>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; vertical-align: top; overflow-wrap: anywhere;"><a style="color: #4664ff;" href="https://github.com/limxdynamics/tron2_openpi">https://github.com/limxdynamics/tron2_openpi</a></td>
    </tr>
    <tr>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; vertical-align: top;">tron2_env</td>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; vertical-align: top;">TRON2 environment package required by <code>tron2_openpi</code> for training and deployment</td>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; vertical-align: top; overflow-wrap: anywhere;"><a style="color: #4664ff;" href="https://github.com/limxdynamics/tron2_env">https://github.com/limxdynamics/tron2_env</a></td>
    </tr>
    <tr>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; vertical-align: top;">Pre-trained weights</td>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; vertical-align: top;">Released model weights, including <code>pi05_tron2_candy</code></td>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; vertical-align: top; overflow-wrap: anywhere;"><a style="color: #4664ff;" href="https://huggingface.co/limxdynamics/tron2-openpi-models">https://huggingface.co/limxdynamics/tron2-openpi-models</a></td>
    </tr>
    <tr>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; vertical-align: top;">Data conversion script</td>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; vertical-align: top;">Converts recorded MCAP data to the LeRobot format</td>
      <td style="border: 1px solid #e0e0e0; padding: 8px 12px; vertical-align: top; overflow-wrap: anywhere;"><a style="color: #4664ff;" href="https://oss.limxdynamics.com/uploads/attachment/2026/09/23/1790164596233_limx_mcap2lerobot.zip">limx_mcap2lerobot.zip</a></td>
    </tr>
  </tbody>
</table>




