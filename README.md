# Paper Reimplementation: Language Models Can Rewrite Their Own Weights

This repository reimplements and extends Fan et al., *Language Models Can Rewrite Their Own Weights* ([arXiv:2611.21885](https://arxiv.org/abs/2611.21885), [project page](https://www.jiajunfan.com/projects/self-rewriting/)). As of November 25, 2026, neither releases code, data splits, edits or transcripts.

In the paper, Qwen3.5-27B edits its own weights by its own reasoning, with no gradients, no optimizer and no stronger model. Its edits make it up to 6.6 points more accurate on competition-math problems it never saw. Every result in the paper is on mathematics.

## Goal

1. **Reimplement and verify.** Rebuild the paper's session and reproduce its math results. See [reimplementation.md](reimplementation.md).
2. **Change the data to coding.** Keep the session fixed, swap in a coding practice set, and review what the model edits and whether it helps. See [change-data.md](change-data.md).
3. **Potentially, new datasets.** Repeat step 2 with other domains, such as physics, if step 2 warrants it.

How every edit is evaluated, on which data, and what counts as a result, is in [eval.md](eval.md).

## Repository layout

| File | Contents |
|---|---|
| [README.md](README.md) | Goal, layout, hardware and environment |
| [reimplementation.md](reimplementation.md) | Rebuilding the session and re-running the paper's math experiment |
| [change-data.md](change-data.md) | Swapping the math practice set for coding data |
| [eval.md](eval.md) | Evaluation data, statistics, controls, hypotheses and pass criteria |
| [commands.md](commands.md) | Linux and Longleaf commands used, with what each did |
| `languagemodels.pdf` | The paper |

## Hardware and environment

All runs use the `rtx6k_gpu` partition of Longleaf, the UNC Research Computing cluster. The details below come from the [Longleaf GPU guide](https://help.rc.unc.edu/gpu/) (updated November 18, 2026) and [Getting Started on Longleaf](https://help.rc.unc.edu/getting-started-on-longleaf) (updated August 17, 2026).

| Item | Setting |
|---|---|
| Cluster | Longleaf, UNC Research Computing |
| Operating system | Red Hat Enterprise Linux 9 |
| Scheduler | Slurm, partition `rtx6k_gpu`, QOS `gpu_rtx6k` |
| GPU | NVIDIA RTX Pro 6000, 96 GB, no NVLink |
| GPUs in the partition | 72 |
| Per-job limits | 16 CPUs and 128 GB of system RAM. More needs approval from research@unc.edu |
| Time limit | 11 days, unless a partition states otherwise |
| Scheduling priority | Fair share, weighted towards GPU use: the more GPU hours a PI group uses, the lower its later jobs rank |
| CUDA | `module load cuda/12.9` |
| Python | `module add anaconda/2024.02` |
| Login node | `longleaf.unc.edu` with your ONYEN, through the campus VPN when off campus |
| Home storage | `/nas/longleaf/home/<onyen>`, 50 GB soft and 75 GB hard limit, backed up |
| Working storage | `/hickory/users/<o>/<n>/<onyen>`, 2 TB and 3,000,000 files, not backed up |

The 27B model's bf16 weights take about 54 GB, so one model copy fits on one GPU. Each job requests one GPU, as the Longleaf guide asks unless a job can use several. Keep model weights, saved edits and results in `/hickory/users`, not in home storage.

A job header within these limits:

```bash
# One task (one program)
#SBATCH --ntasks=1
# 16 CPU cores, the per-job maximum
#SBATCH --cpus-per-task=16
# 128 GB of system RAM, the per-job maximum
#SBATCH --mem=128g
# Stop after 1 day
#SBATCH --time=1-00:00:00
# The RTX Pro 6000 partition
#SBATCH --partition=rtx6k_gpu
# The QOS this partition requires (gpu_access is refused here)
#SBATCH --qos=gpu_rtx6k
# One GPU
#SBATCH --gres=gpu:1
```

The guides do not list CPUs or system RAM for the RTX Pro 6000 nodes.

## Status

Planning. No results yet.

## Citation

```bibtex
@article{fan2026selfrewriting,
  title   = {Language Models Can Rewrite Their Own Weights},
  author  = {Fan, Jiajun and He, Neil and Yang, Changyi and Mao, Andy and Zhu, Xi and Cheng, Chaoran and Liang, Kaiqu and Shen, Shuaike and Li, Zetian and Guo, Ruihan and Liu, Ge},
  journal = {arXiv preprint arXiv:2611.21885},
  year    = {2026}
}
```
