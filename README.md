 # AV_IRL

This project contains utilities for training and evaluating autonomous vehicle agents using inverse reinforcement learning (IRL). It demonstrates adversarial IRL techniques based on **Generative Adversarial Imitation Learning (GAIL)** and **Adversarial Inverse Reinforcement Learning (AIRL)**. Training scripts rely on [Stable-Baselines3](https://github.com/DLR-RM/stable-baselines3), [imitation](https://github.com/HumanCompatibleAI/imitation), and `gymnasium` environments representing highway-style driving tasks.

All environments are wrapped with a **SafeDistanceRewardWrapper** which subtracts a continuous penalty for driving too close to nearby vehicles. Distances to the lead car and lateral neighbours are inferred directly from the environment using the `neighbour_vehicles` and `close_vehicles_to` helpers from `highway-env`. A **TimePenaltyWrapper** applies a small negative reward each step so agents learn to reach their goal quickly. These wrappers encourage safer behaviour and faster completion compared to the previous boolean collision reward.

## Project Layout

```
scripts/                Training and evaluation scripts
    airl_expert_ppo_training.py
    airl_train_loop.py
    gail_train_loop.py
    reward_comparison.py
    final_eval.py
requirements.txt        Python dependencies
LICENSE                 Project license (MIT)
```

## Setup

We recommend using Miniconda to manage the Python environment:

```bash
conda create -n av_irl python=3.10
conda activate av_irl
pip install -r requirements.txt
pip install -e .
```

Alternatively, a standard virtual environment works as well:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
pip install -e .
```

Installing in editable mode ensures the `av_irl` package is importable when running scripts.

## Usage

Run one of the training loops using the provided CLI options:

```bash
python scripts/airl_expert_ppo_training.py --timesteps 100000
```

The evaluation script prints the mean episode score for an expert or trained learner. Model paths can be overridden with command line options:

```bash
python scripts/final_eval.py e h --num-seeds 10 --expert-path model/expert.zip --learner-path model/learner.zip
```

To reproduce the full workflow on a workstation you can run the convenience script:

```bash
nohup bash scripts/run_pipeline.sh &
```

This trains experts for three `(a, b)` settings, generates rollouts in two environments and trains AIRL/GAIL learners using different rollout sizes.


### Coefficients `a` and `b`

`a` controls the weight of the spped.  `b` determines the weight of the safe-distance
penalty applied by `SafeDistanceRewardWrapper`.

### Why compare them for AIRL?

AIRL attempts to recover a reward function that explains the expert
demonstrations.  By training two AIRL models under different `(a, b)` settings
and comparing their predicted rewards, you can check whether AIRL is sensitive
to reward scaling versus safety penalties.  If the estimated reward functions
change noticeably when `a` and `b` are swapped, AIRL has not fully recovered a
reward that is invariant to reward shaping.

The `reward_comparison.py` script prints the predicted reward for a dummy
observation from each model. Examine these values to judge whether the two
estimated reward networks differ significantly.

### Why not compare them for GAIL?

GAIL learns its reward solely from a discriminator that distinguishes expert
and learner trajectories.  It does not use the environment reward, so changing
`a` or `b` would have no direct effect.  Consequently, GAIL is already
invariant to these coefficients and no comparison is needed.

Refer to the source of each script for additional configurable parameters.

## Testing

Run the unit tests with `pytest`:

```bash
pytest -q
```
