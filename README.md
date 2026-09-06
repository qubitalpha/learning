# AI Systems & LLM Engineering: Master Learning Roadmap

## Track 1: LLM Inference & Serving Systems

> **Associated Project**: [`mini-vllm`](../mini-vllm/README.md)  
> **Target Hardware**: 16GB Apple Silicon MacBook Pro  
> **Philosophy**: First-principles physical grounding, bottleneck-driven progression, bare-metal implementation before abstractions.  
> **Milestone Completion Rule**: Before any milestone is marked `- [x]`, the implementation must be tested and you must pass a short concept & edge-case grilling session.

---

### Chapter 0: Physical Grounding and Raw Model Artifacts

Pre-text video: Andrej Karpathy - "Intro to Large Language Models" (1 hour) and "Let's build the GPT Tokenizer" (2 hours).
* What to extract:
  * The physical definition of a model: two files on disk (parameters weight blob and run.c/python code).
  * How Byte-Pair Encoding (BPE) merges character pairs into token IDs.
  * Why special tokens (BOS, EOS) exist and how vocabulary lookup tables work.
* What to ignore:
  * Model training compute cluster management, data scraping, RLHF politics, and building tokenizers for massive web datasets from scratch.

Milestones for Chapter 0:
- [ ] Milestone 1: Download a tiny open-source model (e.g. SmolLM-135M or Qwen2.5-0.5B). Run a script to list and inspect the files in the directory: `config.json`, `tokenizer.json`, and `model.safetensors`.
- [ ] Milestone 2: Load `tokenizer.json` directly in Python. Write code to convert a text sentence into an array of integers and decode integers back to text. Inspect token boundaries.
- [ ] Milestone 3: Open `model.safetensors` using the safetensors library. Iterate over the tensor dictionary to inspect tensor names, shapes, and floating-point data types. Calculate the exact byte size on disk.
- [ ] Milestone 4: Write a bare-metal forward pass script in Python. Pass an array of token IDs, perform matrix multiplication against the loaded weights, and extract the raw array of next-token logits.
- [ ] Milestone 5: Build a naive text generator: take the highest probability logit, convert it to a token, append it to the input array, and loop until an end-of-sequence token is returned.

---

### Chapter 1: The Inference Lifecycle and the KV-Cache Bottleneck

Pre-text video: Andrej Karpathy - "Let's build GPT: from scratch, in code, spelled out" (First 60 minutes only).
* What to extract:
  * How token embeddings are multiplied by Query, Key, and Value projection matrices.
  * The attention formula: Query multiplied by Key transpose gives token affinity scores; multiplying scores by Value produces the output vector.
  * The forward pass execution order from layer 0 to the final linear projection.
* What to ignore:
  * Loss calculation, cross-entropy derivation, backpropagation, optimizer step (`optimizer.step()`), and the training loop on the Shakespeare dataset.

Milestones for Chapter 1:
- [ ] Milestone 6: Feed a 1,000-token prompt into the Milestone 5 naive generator. Measure token generation latency per step and observe how generation slows down quadratically as sequence length grows because the entire prompt is recomputed every token.
- [ ] Milestone 7: Implement a static Key-Value (KV) cache in Python. Modify the attention function to store past Key and Value tensors in an array. Pass only the single newest token into subsequent forward passes. Measure the linear speedup.
- [ ] Milestone 8: Calculate and verify the physical KV-cache memory formula: `2 * 2 bytes * num_layers * num_heads * head_dim * sequence_length`. Run a script that simulates 10 concurrent users with 2,048 tokens context and observe the exact RAM allocation footprint.

---

### Chapter 2: Memory Management and PagedAttention

Pre-text paper / concepts: PagedAttention (vLLM paper: "Efficient Memory Management for Large Language Model Serving with PagedAttention").
* What to extract:
  * Virtual memory concepts applied to GPU tensors: logical blocks versus physical blocks.
  * How static contiguous tensor allocation causes internal and external memory fragmentation.
  * Block tables and copy-on-write mechanics for shared prompt prefixes.

Milestones for Chapter 2:
- [ ] Milestone 9: Expose memory waste. Write a simulation in Python where 8 concurrent requests reserve maximum context memory upfront. Calculate the percentage of allocated memory that sits empty (fragmentation).
- [ ] Milestone 10: Build a block allocator in Python. Split the simulated memory pool into fixed-size physical blocks (e.g. 16 tokens per block). Maintain a block table mapping each request's logical token positions to physical blocks.
- [ ] Milestone 11: Implement prefix sharing. Demonstrate that two distinct requests with identical system instructions point to the same physical memory blocks, freeing memory for additional concurrent users.

---

### Chapter 3: Continuous Batching and Request Scheduling

Pre-text paper / concepts: Orca ("Orca: A Distributed Serving System for Transformer-Based Generative Models").
* What to extract:
  * Why static batching wastes GPU compute (waiting for the longest request in a batch to finish).
  * Iteration-level scheduling: inserting newly arrived requests into the running batch at every token generation step.

Milestones for Chapter 3:
- [ ] Milestone 12: Expose head-of-line blocking. Run a static batch containing one 10-token request and one 500-token request. Measure compute waste while the engine generates empty padding tokens for the finished request.
- [ ] Milestone 13: Build an iteration-level scheduler in Python. Maintain an arrival queue, an active execution batch, and a completion queue. Step the batch forward by 1 token, evict finished requests immediately, and pull waiting requests into vacant slots.
- [ ] Milestone 14: Implement chunked prefill. Break large incoming prompts into smaller chunks (e.g. 256 tokens) so heavy prefill compute does not cause latency spikes for active decoding requests.

---

### Chapter 4: Transport Protocols and Low-Latency Streaming

Pre-text concepts: Server-Sent Events (SSE) spec, gRPC bidirectional streaming, Python `asyncio` queues.

Milestones for Chapter 4:
- [ ] Milestone 15: Wrap the scheduler from Chapter 3 in an asynchronous Python API using FastAPI and `asyncio.Queue`. Expose an HTTP endpoint streaming token deltas via Server-Sent Events (SSE).
- [ ] Milestone 16: Build a benchmarking client. Generate 20 concurrent async HTTP streaming requests and calculate Time to First Token (TTFT) and Inter-Token Latency (ITL) percentiles (p50, p90, p99).
- [ ] Milestone 17: Build a gRPC streaming service interface for the engine. Implement client-side backpressure handling so slow network consumers do not cause unbounded queue growth in the server memory.

---

### Chapter 5: Quantization and Model Compression

Pre-text concepts: Weight-only quantization, scale factors, zero points, FP16 to INT8 conversion.

Milestones for Chapter 5:
- [ ] Milestone 18: Write a Python module that takes an FP16 weight matrix, computes min/max range scale factors, and converts the matrix to 8-bit integers (INT8). Verify that memory usage drops by 50%.
- [ ] Milestone 19: Measure the dequantization overhead. Implement the forward pass step where INT8 weights are cast back to FP16 before matrix multiplication. Compare memory savings against CPU/GPU compute overhead.

---

### Chapter 6: Dynamic Adapters and Multi-Tenancy (LoRA Serving)

Pre-text concepts: LoRA (Low-Rank Adaptation), S-LoRA / Punica architecture.

Milestones for Chapter 6:
- [ ] Milestone 20: Load a base model weights file and a separate small LoRA adapter tensor file (A and B low-rank matrices). Implement the forward pass addition: `output = base_weight(x) + (B * A)(x)`.
- [ ] Milestone 21: Build a multi-tenant adapter dispatcher. Accept concurrent incoming requests where Request 1 asks for Model-Base, Request 2 asks for Model-AdapterA, and Request 3 asks for Model-AdapterB. Route and apply the appropriate adapter matrix dynamically without reloading the 14GB base model.

---

## Future Learning Tracks

As new projects spin out from `learning`, their master curricula and checklists will be indexed here:
- **Track 2**: Post-Training, SFT & RLHF / DPO (Spin-out Project: TBD)
- **Track 3**: Distributed Training & Megatron Parallelisms (Spin-out Project: TBD)
- **Track 4**: GPU Kernel Engineering & Triton / Metal (Spin-out Project: TBD)
- **Track 5**: Agent Runtimes & Tool-Use Execution Engines (Spin-out Project: TBD)

