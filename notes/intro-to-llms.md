# Intro to Large Language Models — Notes & Key Takeaways

- **Source Video**: [Andrej Karpathy - "Intro to Large Language Models" (YouTube)](https://www.youtube.com/watch?v=zjkBMFhNj_g&t=2429s)
- **Status**: Completed (Done)

---

## 1. Physical Anatomy of an LLM: The Two Files

At its fundamental physical layer, a deployed LLM (such as Llama 2 / `llama2.c`) is remarkably minimal—it consists of just **two files**:

1. **The Parameters File (`parameters.bin` / weights blob)**:
   - A binary dump containing billions of numerical values (weights $W$ and biases $b$).
   - Directly connects to the foundational concepts from [`micrograd`](../../micrograd/): every connection has a weight and an associated bias learned via gradient descent.
2. **The Architecture Code (`run.c` / `model.py`)**:
   - The executable code that implements the forward pass.
   - Defines the structural wiring: layers, multi-head self-attention, feed-forward networks, normalization layers, and activation/projection functions (e.g. SwiGLU, GELU, Softmax).

> **Key Insight**: The model architecture and parameters are tightly coupled. The code expects the exact tensor shapes, layer counts, and parameter sequence laid out in the weights file; neither is functional without the other.

---

## 2. Inference vs. Training Dynamics: Memory & Activations

- **Training**: Every forward pass produces intermediate activations ($z = Wx + b$, attention logits, pre-activations). These values must be retained in VRAM/memory throughout the forward pass so they can be referenced during the backward pass to calculate gradients via backpropagation.
- **Inference**: There is no backpropagation, no gradient tracking, and no optimizer step.
  - Once a layer computes its output vectors, intermediate activation values ($Wx + b$) are discarded and the memory is freed up immediately.
  - The only memory that must persist across generation steps is the model weights themselves and the cached past token projections (**KV-cache**).

---

## 3. The 3-Stage Training Pipeline

LLMs undergo a disciplined three-stage progression:

```
[Raw Internet Data] 
       │
       ▼ (Stage 1: Pre-training)
[Base Model] ─────────► Pure text completion / next-token prediction (makemore at scale)
       │
       ▼ (Stage 2: Supervised Fine-Tuning - SFT)
[Assistant Model] ────► Follows instructions, answers questions (Q&A dialogues)
       │
       ▼ (Stage 3: RLHF / Alignment)
[Aligned Model] ──────► Human preference aligned, refusal boundaries, safety guardrails
```

1. **Pre-training (Unsupervised / Self-Supervised)**:
   - Predict the next token across massive text corpuses (the internet).
   - This is the scaled-up realization of what was built from scratch in [`makemore`](../../micrograd/makemore/): compressing statistical world knowledge into token transition probabilities.
   - Outputs a "Base Model" (a document completer, not an assistant).
2. **Supervised Fine-Tuning (SFT)**:
   - Continuing the training process, but on curated, high-quality, human-annotated instruction-response datasets.
   - Shifts the model's behavior from predicting random web text to acting as a conversational assistant.
3. **Reinforcement Learning from Human Feedback (RLHF)**:
   - Scoring responses using human preference rankings and training reward models (or using DPO).
   - Calibrates tone, steers helpfulness, and enforces safety boundaries.

---

## 4. Scaling Laws

- Model performance scales predictably as a power law of two primary dimensions:
  1. **Parameter count** (model capacity / number of weights).
  2. **Data volume** (number of high-quality tokens seen during pre-training).
- When compute (FLOPs) and data are scaled together according to optimal compute ratios (Chinchilla scaling), test loss drops monotonically and more complex reasoning capabilities emerge.

---

## 5. Security & Adversarial Vulnerabilities

Because LLMs ingest natural language as both code (instructions) and data, they are susceptible to unique attack surfaces:

- **Jailbreaking**: Crafting adversarial prompts, hypothetical scenarios, or role-play frames to circumvent safety guardrails.
- **Prompt Injection**:
  - *Direct*: User commands instructing the model to disregard prior system instructions.
  - *Indirect*: Poisoned third-party data sources (e.g. text embedded in images, Google Docs, search results, or web pages) that hijack the model's execution flow.
- **Universal Transferable Suffixes**: Mathematically derived adversarial token strings appended to prompts that reliably bypass safety filters across multiple disparate models.
- **Data Poisoning**: Injecting subtle backdoors or biased data into web scrape datasets or fine-tuning pipelines during training.

---

## 6. The "LLM as an Operating System" Analogy & Roadmap

Karpathy frames the emerging LLM paradigm as an operating system kernel:

| Traditional OS | LLM OS Equivalent | Function |
| :--- | :--- | :--- |
| **CPU** | **LLM Core Engine** | Central reasoning & orchestrator |
| **RAM** | **Context Window** | Immediate working memory |
| **Hard Disk / SSD** | **RAG / Vector DB / Embeddings** | Long-term indexed storage |
| **Peripherals / I/O** | **Tools / Function Calling / APIs** | Web browsing, Python interpreter, terminal |

### Personal Takeaway & Next Steps
- Currently operating largely at the **application layer** (prompting, wrapping APIs, UI).
- The goal is to move down into the **systems and runtime layer**:
  - Understanding raw weight formats (`safetensors`, GGUF, binary blobs).
  - Understanding low-level inference execution, the KV-cache bottleneck, memory paging, and custom serving systems via [`mini-vllm`](../../mini-vllm/README.md).

