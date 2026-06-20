
# Towards Embodiment Scaling Laws in Robot Locomotion

We uncover embodiment scaling laws: training on diverse robot embodiments enables broad generalization to unseen ones, demonstrated in a locomotion study across ~1,000 robots.

https://github.com/user-attachments/assets/757c505f-690a-4dc3-b996-f062b88c5bcf


**[Website](https://embodiment-scaling-laws.github.io/) |  [Paper](https://arxiv.org/pdf/2505.05753) | [Models (RL Experts)](https://drive.google.com/file/d/16UgSzShxdfoj8ZT_WemAawXZq-wXLf7l/view?usp=sharing)**

If you find the codebase useful, please consider citing:
```
@article{ai2025towards,
  title={Towards Embodiment Scaling Laws in Robot Locomotion},
  author={Ai, Bo and Dai, Liu and Bohlinger, Nico and Li, Dichen and Mu, Tongzhou and Wu, Zhanxin and Fay, K and Christensen, Henrik I and Peters, Jan and Su, Hao},
  journal={Conference on Robot Learning (CoRL)},
  url={https://arxiv.org/abs/2505.05753},
  year={2025}
}
```

## Installation
- Install Isaac Lab, see
  the [installation guide](https://isaac-sim.github.io/IsaacLab/main/source/setup/installation/pip_installation.html).

- Install the library `embodiment_scaling_laws` and our customized version of `rsl_rl`
```
cd embodiment-scaling-law
pip install -e exts/embodiment_scaling_laws/
pip install -e rsl_rl/
```

## Expert Training
This section introduces how to train expert policy.

### 0. Set up `GenBot1K`
Unzip our robot configure dataset `GenBot1K` to `exts/embodiment_scaling_laws/embodiment_scaling_laws/assets/Robots/GenBot1K-v7`: 
```
cd exts/embodiment_scaling_laws/embodiment_scaling_laws/assets/Robots
unzip GenBot1K.zip
mv GenBot1K GenBot1K-v7
```

### 1. RL Policy Training
To train just one robot, run 
```angular2htmlpyt
python scripts/expert/train.py --task Gendog1
```
If you do not wish to have the visualization open, which could slow down training significantly, add `--headless` to the command. 

To train multiple robots in sequence, run
```angular2html
bash scripts/expert/train_batch.sh --tasks Gendog1 Gendog2 Gendog3 Gendog4 Gendog5
```

File `exts/embodiment_scaling_laws/embodiment_scaling_laws/tasks/configs/environment/__init__.py` contains all the task names that we can run. For example, for `GenBot1k`, we can run `Gendog{i}` where `i` could range from `0` to `331`. 

Optionally: We provide the well trained checkpoints [here](https://drive.google.com/file/d/16UgSzShxdfoj8ZT_WemAawXZq-wXLf7l/view?usp=sharing)(size=5G).

### 2. RL Policy Evaluation
To evaluate and visualize the trained policy, run
```angular2html
python scripts/expert/play.py --task Gendog1
```

## Policy Distillation
This section introduces how to perform teacher-student policy distillation. 

### 0. Data Collection using RL Policies
To generate robot locomotion dataset, data_collection`. For example, run
[//]: # (We first generate a dataset from the teacher policy. The input data is suitable for URMA model, which includes joint description, joint state and general state. The output follows the normal action pattern. Replicate this process for each of the robots.)

```angular2html
python scripts/distillation/play_collect_data.py --task Gendog1 --steps 1000 --resume_path logs/rsl_rl/Gendog1/{date_time}/{checkpoint_file} --reward_log_file reward_log_file.json --headless
```
The detailed return information and observation-action pair dataset is stored in `logs/rsl_rl/{robot_name}/{date_time}/h5py` folder. Please beware that the size of dataset for each robot is 1.3G-7G, and the total size of the dataset is roughly 4T.

For complete dataset collection, check ready-to-go scripts in `scripts/distillation/data_collection`.

### 1. Distillation
To train a student policy, we load the datasets from all robots and perform supervised learning.

For complete policy distillation, check ready-to-go scripts in `scripts/distillation/urma_distillation`.

Please note that part of the dataset is cached in RAM to minimize I/O bottlenecks. A machine with 128 GB of RAM should be sufficient to run all commands. Reducing `--max_files_in_memory` will decrease the RAM requirement roughly linearly, but at the cost of relying on more localized gradient estimates, which may degrade model performance.


### 2. Distilled Policy Evaluation 

Finally, to load the student policy for evaluation, run
```angular2html
python scripts/distillation/eval_student_model_urma.py --task Gendog1 --ckpt_path log_dir/{policy_name}/{checkpoint_file} --model urma  --log_file reward_log_file.json
```
Detailed return information will be saved at reward_log_file.json.

For complete evaluation on test set, check ready-to-go scripts at `scripts/distillation/urma_evaluation/eval_student_model_urma_test_set.sh`.
