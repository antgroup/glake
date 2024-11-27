## GLake: Optimizing GPU memory management & IO transmission
[中文](docs/readme_cn.md)
## Latest News
- [2024.10] We release LayerKV [arxiv](https://arxiv.org/abs/2410.00428), efficient CPU-GPU KV Cache management to decrease TTFT.
- [2024.07] We release vTensor, our LLM serving and KV Cache management system using VMM technique.
- [2024.05] GLake overview and recent update is presented on AICon 2024 (in Beijing, China, 2024-05-17) [here](https://github.com/intelligent-machine-learning/glake/blob/main/docs/%E8%9A%82%E8%9A%81-GLake%E6%98%BE%E5%AD%98%E4%B8%8E%E4%BC%A0%E8%BE%93%E4%BC%98%E5%8C%96-AIConf-V1.0.pdf)
- [2024.05] The presentation slides in ASPLOS'24 can be found [here](https://docs.google.com/presentation/d/1ZM7OfvOMc5zZ2oIGJONhs_gFqsMLjTogNtD0I59TNfE/edit#slide=id.p1)
- [2024.05] Preview: **serverless** with on-demand GPU memory shink (ckpt to CPU while), keeping process alive and auto backfill upon request coming
- [2024.04] Preview: optimizing **LLM KV cache**, different than PagedAttention, flexible and easy-to-adopt 
- [2024.04] Preview: **cross-stream** GPU memory optimization dev done, will be released soon;  integration with PyTorch expand segment is in-progress by AntGroup and Alibaba contributor
- [2024.03] compatible with PyTorch-1.13.1 at branch torch-1.13.1, also with a whl package
- [2023.11.10] Our paper: [GMLake: Efficient and Transparent GPU Memory Defragmentation for Large-scale DNN Training with Virtual Memory Stitching](https://arxiv.org/abs/2401.08156), just accepted by **ASPLOS24**
- [Blog](docs/readme_cn.md)

## Introduction
AI large model training and inference are increasingly confronted with the challenges of memory wall and IO transmission wall, that is, the growth of GPU memory capacity and IO bandwidth cannot keep up with the growth rate of AI model size. 

To address these challenges, GLake is an acceleration library and relevant utilites that work at the bottom layer (GPU virtual and physical memory management) and system layer (multi-GPU, multi-path, and multi-tasking) to optimize GPU memory and IO. 

GLake enables AI training, inference (including converting large models to TensorRT or ONNX Runtime on NVIDIA A10/3090) and DevOps (like Notebook) to fully utilize the underlying hardware resources, improving training throughput by up to 4 times, saving inference memory by up to 3 times, and accelerating IO transmission by 3~12 times. 

To use GLake, the simplest way is to replace the underlying library (e.g., libcuda.so or PyTorch libc10_cuda.so), though more graceful way is to follow the detailed steps.

### Motivation
- **GPU memory bottleneck** GPUs are known for their high computing power and high concurrency. As a peripheral, however, its memory capacity (currently 80GB for mainstream training GPUs A100 and 24GB for mainstream inference GPUs A10) still restricts the use of its computing power. Especially recently, the growing demand for GPU memory capacity for large models has been much higher than the hardware development of GPU memory.
- **IO transmission bottleneck** Comparing the GPU computing power and CPU-GPU IO bandwidth of various GPU generations, it is not difficult to find that the limitations of transmission wall are intensifying and will be unlikely to be solved in the short term. It is worth noting that based on the customized interconnection NVLink, the GPU-to-GPU bandwidth is significantly faster than the PCIe bandwidth. In addition, GPU memory bandwidth (HBM, GDDR) is a performance bottleneck of large model inference.
### Architecture
GLake is designed with a layered architecture. Currently tests and verfications focus on PyTorch and NVIDIA GPUs, we're working on more devices support:

<div align="center">
<img src="docs/figures/glake_arch_en.png" alt="Editor" width="700">
</div>

- **Hardware interface** includes GPUs and interconnection, currently mainly based on NV GPU (supporting NVLink, P2P, Unified Addressing, VMM, IPC, etc.). The interface is adapting to support domestic AI cards, and will consider supporting new interconnections (such as CXL) in the future.
- **GPU memory pool** provides global and heterogeneous GPU memory pools, built-in GPU memory fragmentation optimization, multi-stream and multi-process memory reuse, and memory security.
- **Core optimization layer** provides value-added optimization functions, including global allocation, multi-channel concurrency, tiering, memory deduplication, KV-cache optimization, etc.
- **Extension layer** combines the DL framework and the team's self-developed VGPU, which provides integration solutions or extensions, such as PyTorch.
- **Application and ecology** currently focuses on AI training and inference. In the future, different application scenarios can be supported, such as graph computing and graphic rendering.

### Features
- **Efficient**: With internal two-layer GPU memory management and global (multi-GPU, multi-task) optimization, GPU memory pooling, sharing and tiering are realized to provide larger GPU memory for training and inference. Multi-path can accelerate CPU-GPU transmission by 3~12X.
- **Easy to use**: The core functions are transparent to the model and do not require code modification for training and inference, as GLake can be easily plugged into existing engines (such as PyTorch). Meanwhile GPU memory internal stats. (e.g., fragmentation) can be queried online with RPC interface 
- **Open and easy to extend**: Configurable strategies (e.g., compression, data verification, different levels of security check) will be provided.
- **Security**: For troubleshooting problems such as GPU memory out-of-bounds, a built-in GPU memory out-of-bounds detection mechanism will assist in diagnose.
### Quick Results
1. GLake reduces the memory fragmentation by up to 27%, save 25G of GPU memory, and increase the training throughput of a 10B model by up to nearly 4 times.
2. For inference, GLake supports cross-process and cross-model elimination of duplicate memory, saving 3 times memory.
3. GLake accelerates CPU-GPU IO transmission by 3 times.


## Examples
[GMLake tutorial](GMLake/docs/GMLake-tutorial.md)\
[Multi-path tutorial](MultiPath/README.md)
## How it works
- **GMLake** When there is no contineous free buffer to satisfy allocation requests, GMLake will return a complete buffer to users by combining multiple memory fragementation.

<div align="center">
<img src="docs/figures/gmlake.png" alt="Editor" width="500">
</div>

- **Multi-path** CPU-GPU IO throughput is improved by exploiting multiple transfer paths concurrently.

<div align="center">
<img src="docs/figures/multi_path_view.png" alt="Editor" width="700">
</div>

- **Data deduplication** For AI inference, GLake is able to automatically find out duplicate memory use and share them between processes in fine-grained memory.

<div align="center">
<img src="docs/figures/dedup1.png" alt="Editor" width="500">
</div>

## Roadmap
We are working on a few interesting featues. Any questions, suggestions and participations are welcomed. 
- **LLM KV cache** : tackle LLM inference KV cache fragmentation in a unifed and efficient way (a little different from vLLM) 
- **cache-prefetch**: optimize offloading & prefetching in fine-tuning and inference (i.e., atop DeepSpeed, may deep into L2 cache)  
- **tiering**: manage and optimize memory allocations and data moving across cards/nodes and various memory types 
- **data deduplication**: keep single unique content copy in fine-grained block across model instances and processes in inference or serverless 
- **memory debugging**: enable efficient and friendly GPU memory debugging in case of overflow, segmentfault etc
- **more accelerators**: okey, we'll (have to) need more choices
- **more scenarios**: such as GNN、GraphDB etc

## Community
WeChat-Group: in case of invalid, pls contact wx: 13051161982
<div align="left">
<img src="docs/figures/gmlake-wechat.png" alt="Editor" width="500">
</div>
