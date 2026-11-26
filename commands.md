# Commands

Linux and Longleaf commands used in this project, in the order they first came up. Replace `<o>`, `<n>` and `<onyen>` with the first letter of your ONYEN, its second letter and the full ONYEN.

## Connecting and files

| Command | What it does |
|---|---|
| `ssh <onyen>@longleaf.unc.edu` | Logs in to the Longleaf login node |
| `ls -ld /hickory/users/<o>/<n>/<onyen>` | Checks that the working storage folder exists and who owns it |
| `df -h /hickory/users/<o>/<n>/<onyen>` | Shows the free space on the shared Hickory filesystem |
| `du -sh /hickory/users/<o>/<n>/<onyen>/hf` | Shows the total size of the downloaded models |
| `du -sh /hickory/users/<o>/<n>/<onyen>/hf/hub/*` | Shows the size of each downloaded model separately |
| `nano <file>` | Opens a file in a simple text editor (`Ctrl+O` saves, `Ctrl+X` quits) |
| `rm <file>` | Deletes a file for good |
| `grep <text> <file>` | Prints the lines of a file that contain the text |
| `grep --count <text> <file>` | Counts the lines of a file that contain the text |
| `exit` | Closes the current shell. On a GPU node, this ends the job |

## Shell settings

| Command | What it does |
|---|---|
| `export HF_HOME=/hickory/users/<o>/<n>/<onyen>/hf` | Sets where Hugging Face saves models and datasets (kept in `~/.bashrc`) |
| `export CONDA_PKGS_DIRS=...` and `export PIP_CACHE_DIR=...` | Keep conda and pip downloads off the 50 GB home folder (kept in `~/.bashrc`) |
| `source ~/.bashrc` | Loads the settings into the current shell |
| `echo $HF_HOME` | Prints a variable's value, to check it is set |
| `which python` | Shows which Python runs when you type `python` |

## Software and the Python environment

| Command | What it does |
|---|---|
| `module add anaconda/2024.02` | Loads Longleaf's Anaconda, which provides `conda` |
| `module load cuda/12.9` | Loads the CUDA toolkit (optional, silences the DeepGEMM warning) |
| `conda create --prefix /hickory/users/<o>/<n>/<onyen>/selfrewrite/env python=3.12` | Creates the project's Python environment on Hickory |
| `conda activate /hickory/users/<o>/<n>/<onyen>/selfrewrite/env` | Turns the environment on |
| `conda deactivate` | Turns the environment off, clearing a stale one copied to a GPU node |
| `source /nas/longleaf/rhel9/apps/anaconda/2024.02/bin/activate <env path>` | Turns the environment on when `conda activate` doesn't work |
| `pip install vllm` | Installs vLLM, PyTorch and the CUDA libraries |
| `pip show vllm` | Shows the installed vLLM version (0.34.0) |
| `python -m pip show vllm` | Checks that this exact Python can see vLLM |
| `python -c "import vllm; print(vllm.__version__)"` | Checks that vLLM imports |
| `python -c "import torch; print(torch.cuda.get_device_name(0))"` | Checks that PyTorch can see the GPU |
| `hf download Qwen/Qwen3.5-27B` | Downloads the model to `HF_HOME` |
| `python <script>.py` | Runs a Python script |
| `python <script>.py > <file>.log 2>&1` | Runs a script and saves all its output, errors included, to a log file |

## Slurm (getting and checking GPUs)

| Command | What it does |
|---|---|
| `sacctmgr show association user=<onyen> format=account%30,partition,qos%60` | Lists the QOS levels the account can use (this is how `gpu_rtx6k` was found) |
| `sacctmgr show qos format=name` | Lists every QOS on Longleaf |
| `srun --partition=rtx6k_gpu --qos=gpu_rtx6k --gres=gpu:1 --cpus-per-task=16 --mem=128g --time=1:00:00 --pty bash` | Gets an interactive shell on a GPU node for one hour |
| `srun --jobid=<jobid> --overlap --pty bash` | Opens a second shell inside a running job |
| `squeue --me` | Lists your jobs and whether they are running or waiting |
| `squeue --me --start` | Shows when waiting jobs are expected to start |
| `squeue --partition=rtx6k_gpu` | Lists everyone's jobs on the partition |
| `sinfo --partition=rtx6k_gpu` | Shows which nodes in the partition are idle, partly used or full |
| `scancel <jobid>` | Cancels a job |
| `seff <jobid>` | Shows how much of the requested CPU and RAM a finished job used |
| `sacct --jobs=<jobid> --format=JobID,State,Elapsed,Timelimit,ReqMem,MaxRSS,AllocCPUS,TotalCPU` | Shows a finished job's time, memory and CPU use |
| `sacct --starttime=<date> --format=JobID,JobName,State,Elapsed` | Lists your jobs since a date |
| `nvidia-smi` | Shows the GPU, its memory use and the programs on it |
| `nvidia-smi --loop=2` | Reprints that every 2 seconds |
| `nvidia-smi --query-gpu=utilization.gpu,memory.used,memory.total --format=csv --loop=10` | Prints GPU use and memory as one line every 10 seconds |
