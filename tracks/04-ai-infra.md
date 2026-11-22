# Track 4: AI infra

LLM inference infrastructure: GPU kernels, serving engines, distributed serving. The systems foundations carry over from trading work; the gap is **GPUs and model internals**.

The live week-by-week plan is [`ROADMAP.md`](../ROADMAP.md). Breadth week 8 is a toy CUDA kernel. Week 9 runs vLLM or llama.cpp. The phase lengths below are the old depth-first layout and remain the catalog for AI specialization after week 13. Keep ticking checkboxes here.

**Trading-systems overlap:** the inference request path is an order path. Gateway → queue → pinned scheduler → GPU worker is the same pipeline as modern_exchng_cpp, with the same problems: tail latency, batching against latency, allocation-free hot paths.

---

## Phase A: Model internals and the cost model (3 weeks, Q2)

- [ ] Transformer forward pass: embeddings, RMSNorm, attention (MHA → GQA), RoPE, SwiGLU MLP, sampling
- [ ] Why **prefill is compute-bound and decode is memory-bandwidth-bound**, and what that means for batching
- [ ] KV cache size: `2 × layers × kv_heads × head_dim × bytes × tokens`; work it out for Llama-3-8B at 8K context
- [ ] Roofline: arithmetic intensity, ridge point, where GEMM, attention and decode sit
- [ ] **Build:** a single-threaded C++ forward pass for a tiny Llama-style model (e.g. stories15M), checked token-for-token against PyTorch

**Done when:** I can explain from memory why batch-1 decode on an H100 is far below peak FLOPs.

## Phase B: GPU architecture and CUDA (9 weeks, Q2)

- [ ] Execution model: SMs, warps, SIMT, divergence, occupancy, latency hiding
- [ ] Memory hierarchy: HBM → L2 → shared memory/L1 → registers; coalescing; bank conflicts
- [ ] Profiling: Nsight Systems (timeline) and Nsight Compute (roofline, stall reasons)
- [ ] Tensor cores: WMMA/MMA, then CUTLASS/CuTe concepts
- [ ] Async copies (`cp.async`, TMA on Hopper), pipelining, warp specialization
- [ ] Triton: write the same kernels in Triton, compare effort against speed
- [ ] → SI-04 SGEMM ladder, SI-05 fused kernels

**Done when:** SI-04 reaches ≥ 80% of cuBLAS for FP32 SGEMM at 4096², and I can name the limiter from a Nsight Compute report.

**Overlap:** coalescing is cache-line-aware layout, shared memory is a software-managed L1, warp divergence is the branch mispredict of SIMT.

## Phase C: Inference engine (12 weeks, Q4) and upstream work (Q5)

- [ ] Weight loading (safetensors, mmap), tokenizer integration
- [ ] KV cache: contiguous, then **paged** (block tables, block allocator)
- [ ] Scheduling: static → **continuous batching**; prefill/decode split; chunked prefill
- [ ] Weight-only quantization (INT8, then INT4 / AWQ-style)
- [ ] Prefix caching (radix tree, the SGLang idea)
- [ ] Speculative decoding (draft model or Medusa/EAGLE-style heads)
- [ ] CUDA graphs for decode
- [ ] OpenAI-compatible HTTP endpoint and load generator (reuse SI-01)
- [ ] → SI-09 spinfer
- [ ] Profile vLLM / SGLang / llama.cpp: GPU with Nsight Systems; CPU side (scheduler loop, detokenization, Python) with `py-spy`, `perf` and triangulator
- [ ] → SI-12 upstream perf PR

**Done when:** spinfer has a TTFT / TPOT / throughput / p99 table against llama.cpp and vLLM on the same GPU, and one perf PR is merged upstream.

## Phase D: Distributed serving (11 weeks, Q6)

- [ ] Collectives: all-reduce, all-gather, reduce-scatter; ring vs tree; NCCL and `nccl-tests`
- [ ] Interconnects: NVLink, PCIe, InfiniBand/RoCE, RDMA, GPUDirect
- [ ] Parallelism for inference: tensor, pipeline, expert (MoE), data
- [ ] Disaggregated prefill/decode, KV-cache transfer between nodes
- [ ] Routing: prefix-cache affinity, load-aware placement, autoscaling signals
- [ ] → SI-13 tensor parallelism, SI-14 cache-aware router

**Done when:** measured scaling efficiency for SI-13, and SI-14 beats round-robin on a shared-prefix workload.

## Optional: Training infra

FSDP/ZeRO, activation checkpointing, Megatron-style 3D parallelism, checkpointing at scale, failure detection on large clusters. Only if a target role needs it.

---

## Resources

**Model internals**
- ★ Andrej Karpathy, [Neural Networks: Zero to Hero](https://karpathy.ai/zero-to-hero.html) and [llm.c](https://github.com/karpathy/llm.c)
- Horace He, [Making Deep Learning Go Brrrr From First Principles](https://horace.io/brrr_intro.html)
- [How to Scale Your Model](https://jax-ml.github.io/scaling-book/)

**GPU and CUDA**
- ★ Hwu, Kirk, El Hajj, *Programming Massively Parallel Processors* (4th ed.)
- ★ Simon Boehm, [How to Optimize a CUDA Matmul Kernel](https://siboehm.com/articles/22/CUDA-MMM)
- ★ [GPU MODE lectures](https://github.com/gpu-mode/lectures)
- NVIDIA [CUDA C++ Programming Guide](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html) and [Best Practices Guide](https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/index.html)
- [Triton tutorials](https://triton-lang.org/main/getting-started/tutorials/index.html), [CUTLASS](https://github.com/NVIDIA/cutlass)

**Inference**
- ★ Orca, continuous batching (OSDI '22)
- ★ [FlashAttention](https://arxiv.org/abs/2205.14135), [FA-2](https://arxiv.org/abs/2307.08691), [FA-3](https://arxiv.org/abs/2407.08608)
- ★ [PagedAttention / vLLM](https://arxiv.org/abs/2309.06180)
- [SGLang / RadixAttention](https://arxiv.org/abs/2312.07104); AWQ and GPTQ papers
- Code: [llama.cpp](https://github.com/ggml-org/llama.cpp), [vLLM](https://github.com/vllm-project/vllm), [SGLang](https://github.com/sgl-project/sglang), and nano-scale engines (nano-vllm, mini-sglang)

**Distributed**
- ★ [Megatron-LM](https://arxiv.org/abs/1909.08053), [DistServe](https://arxiv.org/abs/2401.09670), Splitwise
- [nccl-tests](https://github.com/NVIDIA/nccl-tests); Mooncake; NVIDIA Dynamo and llm-d as production references
- Training: Hugging Face, [The Ultra-Scale Playbook](https://huggingface.co/spaces/nanotron/ultrascale-playbook)

## Hardware

- Phases A to C: one NVIDIA GPU (a consumer RTX card, Ampere or newer). Rent an A100/H100 by the hour for production-like numbers.
- Phase D: rent a multi-GPU node for focused, scripted sessions.
