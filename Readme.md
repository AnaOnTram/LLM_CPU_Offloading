## Background
My PC configuration has always been quite wired. Somehow, I persuaded myself I need to build an <strong> itx </strong> to suit my tiny apartment at Hong Kong. As the 30 series has been natoriously recognized as power hungry. The 3090 Ti draws astonishingly high power with rated TDP of 450W that basically turns my itx build into an oven (<i> At least my cat loves it</i>). Although my CPU was a MODT chip from AMD (Ryzen R9 7945HX), which suppose to have a mobile level TDP, it also suffers from severe overheat issues. Fans makes unbearable noise when the system was under heavy load and the temp constantly bumpted to over 90 degrees for both the CPU and the GPU. Hence, I decided to swift to something that is <strong>"energy efficient" </strong>.
<br>Recently, I "upgraded" my "workstation"'s GPU from a RTX3090 Ti to a [RTX Pro 2000 Blackwell](https://www.nvidia.com/en-us/products/workstations/professional-desktop-gpus/rtx-pro-2000/$0). It was professional grade entry level card that has just been published by NVIDIA with a TDP of 70Watts. Eventhough I want the Pro 4000 sff badly, I don't think anyone has it on hand now. Thus, I was limited to 16GB of GDDR7 VRAM (from previous 24GB GDDR6X). The weaker GPU means having the model load solely on GPU was not feasible and ideal. Thus, I start to explore solutions that might utilize the 16-core Ryzen CPU.

### My current Config
* CPU: <strong> Ryzen 9 7945HX </strong> <i>(16 Cores 32 Threads)</i>
* GPU: <strong>NVIDIA RTX PRO 2000 </strong> <i>(A 5060Ti but running at 70W)</i>
* Mother Boad: <strong> Minisforum BD790i </strong>
* RAM: <strong> 2 * Mircon 48GB 5600 Sodimm </strong> <i>(running at 5200)</i>
* Storage: <strong>SN850 2TB </strong>
* PSU: 850W SFX
<br>System Info
```bash
            .-/+oossssoo+/-.               ross@ANA 
        `:+ssssssssssssssssss+:`           -------- 
      -+ssssssssssssssssssyyssss+-         OS: Ubuntu 24.04.3 LTS x86_64 
    .ossssssssssssssssssdMMMNysssso.       Host: MotherBoard Series 1.0 
   /ssssssssssshdmmNNmmyNMMMMhssssss/      Kernel: 6.14.0-37-generic 
  +ssssssssshmydMMMMMMMNddddyssssssss+     Uptime: 3 hours, 11 mins 
 /sssssssshNMMMyhhyyyyhmNMMMNhssssssss/    Packages: 2523 (dpkg), 20 (snap) 
.ssssssssdMMMNhsssssssssshNMMMdssssssss.   Shell: bash 5.2.21 
+sssshhhyNMMNyssssssssssssyNMMMysssssss+   Resolution: 3840x2560 
ossyNMMMNyMMhsssssssssssssshmmmhssssssso   Terminal: /dev/pts/2 
ossyNMMMNyMMhsssssssssssssshmmmhssssssso   CPU: AMD Ryzen 9 7945HX with Radeon Graphics (32) @ 5.462GHz 
+sssshhhyNMMNyssssssssssssyNMMMysssssss+   GPU: NVIDIA 01:00.0 NVIDIA Corporation Device 2d30 
.ssssssssdMMMNhsssssssssshNMMMdssssssss.   GPU: AMD ATI 06:00.0 Raphael 
 /sssssssshNMMMyhhyyyyhdNMMMNhssssssss/    Memory: 47572MiB / 94247MiB 
  +sssssssssdmydMMMMMMMMddddyssssssss+
   /ssssssssssshdmNNNNmyNMMMMhssssss/                              
    .ossssssssssssssssssdMMMNysssso.                               
      -+sssssssssssssssssyyyssss+-
        `:+ssssssssssssssssss+:`
            .-/+oossssoo+/-.
```
Driver and NVCC info
```bash
#nvcc
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2025 NVIDIA Corporation
Built on Wed_Jan_15_19:20:09_PST_2025
Cuda compilation tools, release 12.8, V12.8.61
Build cuda_12.8.r12.8/compiler.35404655_0
#nvidia-smi
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 580.95.05              Driver Version: 580.95.05      CUDA Version: 13.0     |
+-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA RTX PRO 2000 Blac...    Off |   00000000:01:00.0 Off |                  Off |
| 30%   28C    P8              6W /   70W |    3149MiB /  16311MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
```

# [llama.cpp](https://github.com/ggml-org/llama.cpp.git)
The old classic llama.cpp which needs no further explaination.
### Build Method
```bash
# Native cuda built
cmake -B build -DGGML_CUDA=ON
cmake --build build --config Release -j
```
### Model to be served
[unsloth/Qwen3-Next-80B-A3B-Instruct-GGUF](https://huggingface.co/unsloth/Qwen3-Next-80B-A3B-Instruct-GGUF$0) with Q4_K_XL quantization.
<br> Serving command
```bash
./llama.cpp/build/bin/llama-server -hf  unsloth/Qwen3-Next-80B-A3B-Instruct-GGUF:Q4_K_XL --jinja  -fa on -c 30000 -ngl 999 -ot ".ffn_.*_exps.=CPU" --override-tensor "blk.([0-9]).ffn.*shexp.=CUDA0,.ffn.*shexp.*=CUDA0,ffn_.*_exps.=CPU" --host 0.0.0.0 --threads 32 --api-key 123456 --port 8000
```
### Idle System Resources Usage
* System RAM Usage: 43GB
* VRAM Usage: 3044MiB
* GPU Idle Power: 5W
* Temp: CPU peak: 38°C (IO), GPU: 28°C 
## Performance
```bash
prompt eval time =    6216.65 ms /  1039 tokens (    5.98 ms per token,   167.13 tokens per second)
       eval time =     473.01 ms /    13 tokens (   36.39 ms per token,    27.48 tokens per second)
      total time =    6689.66 ms /  1052 tokens
```
