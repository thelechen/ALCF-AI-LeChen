# inference for Meta-Llama-3-8B-Instruct

## Keywords
- vllm
- one node (4 GPUs)

## Steps
### Request a GPU Compute Node (interactive)
```bash
qsub -I -A <project> -q debug \
-l select=1 \
-l walltime=01:00:00 \
-l filesystems=home:eagle
```

### Conda Environment (first time)
```bash
module use /soft/modulefiles/
module load conda
conda create -n vllm_v071_env python==3.11.9 -y 
conda activate vllm_v071_env 
module use /soft/spack/base/0.8.1/install/modulefiles/Core
module load gcc
pip install vllm
```


### Environment Setup (every session)
```bash
# every time after getting a compute node
module use /soft/modulefiles
module load conda
conda activate vllm_v071_env
module use /soft/spack/base/0.8.1/install/modulefiles/Core
module load gcc
export RAY_TMPDIR="/tmp"
export RAYON_NUM_THREADS=4
export RUST_BACKTRACE=1
export PROMETHEUS_MULTIPROC_DIR="/tmp"
export VLLM_RPC_BASE_PATH="/tmp"
export no_proxy="127.0.0.1,localhost"
```
### HF Setup 
```bash
# skip if you have your settings
export HF_DATASETS_CACHE="/eagle/argonne_tpc/model_weights/"
export HF_HOME="/eagle/argonne_tpc/model_weights/"
export HF_TOKEN="" #Add your token
```
### Run vllm Serve
- run in front (for test)
```bash
vllm serve Qwen/QwQ-32B \
--host 127.0.0.1 \
--tensor-parallel-size 4 \
--gpu-memory-utilization 0.9 \
--enforce-eager #For online serving
```
- run in background
```bash
cd /path/to/logs
nohup vllm serve Qwen/QwQ-32B \
--host 127.0.0.1 \
--tensor-parallel-size 4 \
--gpu-memory-utilization 0.9 \
--enforce-eager &
```
### Start inference

```bash
curl http://127.0.0.1:8000/v1/completions \
  -H "Content-Type: application/json" \
  -d '{
        "model": "Qwen/QwQ-32B",
        "prompt": "Tell me a short joke about HPC.",
        "max_tokens": 1000,
        "temperature": 0.7
      }'
```