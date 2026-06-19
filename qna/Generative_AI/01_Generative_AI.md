# Generative AI – Interview Questions & Answers

**Question**: Explain the Transformer architecture and its core components.

**Answer**: The Transformer consists of an encoder and a decoder, each built from stacked self-attention and feed-forward layers. The encoder processes the input sequence bidirectionally, while the decoder generates output autoregressively. Key components include multi-head attention, positional encoding, layer normalization, and residual connections.

```python
import torch.nn as nn

class TransformerBlock(nn.Module):
    def __init__(self, d_model, n_heads):
        super().__init__()
        self.attention = nn.MultiheadAttention(d_model, n_heads)
        self.ffn = nn.Sequential(
            nn.Linear(d_model, d_model * 4),
            nn.ReLU(),
            nn.Linear(d_model * 4, d_model)
        )
        self.norm1 = nn.LayerNorm(d_model)
        self.norm2 = nn.LayerNorm(d_model)

    def forward(self, x):
        attn_out, _ = self.attention(x, x, x)
        x = self.norm1(x + attn_out)
        ffn_out = self.ffn(x)
        return self.norm2(x + ffn_out)
```

---

**Question**: What is the difference between self-attention and cross-attention?

**Answer**: Self-attention computes attention over the same sequence, allowing each token to attend to every other token within that sequence. Cross-attention computes attention from one sequence (e.g., decoder hidden states) to another (e.g., encoder outputs), enabling the decoder to attend to the input. In Transformers, encoder layers use only self-attention, while decoder layers use masked self-attention followed by cross-attention with encoder outputs.

---

**Question**: How does multi-head attention work and why is it beneficial?

**Answer**: Multi-head attention runs multiple scaled dot-product attention operations in parallel, each with different learned projections, then concatenates the results. This allows the model to jointly attend to information from different representation subspaces at different positions. Each head can learn different linguistic patterns (e.g., syntax, semantics, coreference).

```python
class MultiHeadAttention(nn.Module):
    def __init__(self, d_model, n_heads):
        super().__init__()
        self.n_heads = n_heads
        self.d_k = d_model // n_heads
        self.W_q = nn.Linear(d_model, d_model)
        self.W_k = nn.Linear(d_model, d_model)
        self.W_v = nn.Linear(d_model, d_model)
        self.W_o = nn.Linear(d_model, d_model)

    def forward(self, q, k, v, mask=None):
        B, T, _ = q.shape
        Q = self.W_q(q).view(B, T, self.n_heads, self.d_k).transpose(1, 2)
        K = self.W_k(k).view(B, T, self.n_heads, self.d_k).transpose(1, 2)
        V = self.W_v(v).view(B, T, self.n_heads, self.d_k).transpose(1, 2)
        scores = Q @ K.transpose(-2, -1) / (self.d_k ** 0.5)
        if mask is not None:
            scores = scores.masked_fill(mask == 0, float('-inf'))
        attn = scores.softmax(dim=-1)
        out = (attn @ V).transpose(1, 2).reshape(B, T, -1)
        return self.W_o(out)
```

---

**Question**: Explain absolute vs relative positional encoding.

**Answer**: Absolute positional encoding adds a fixed or learned vector to each token embedding based on its absolute position, which works well for fixed-length sequences. Relative positional encoding encodes the offset between token pairs in the attention computation, enabling better generalization to longer sequences not seen during training. Relative encoding is used in models like GPT-NeoX, LLaMA, and RoFormer.

---

**Question**: What are the key architectural differences between GPT, LLaMA, and Claude?

**Answer**: GPT uses a decoder-only Transformer with causal masking, learned positional embeddings, and LayerNorm after each sublayer (pre-norm). LLaMA introduces RMSNorm instead of LayerNorm, SwiGLU activation in FFN, and rotary positional embeddings (RoPE) for better length generalization. Claude (Anthropic) builds on Transformer architecture with a focus on constitutional AI alignment, large context windows, and extensive RLHF.

---

**Question**: Distinguish pre-training, fine-tuning, and RLHF.

**Answer**: Pre-training trains a model on a large unlabeled corpus using unsupervised objectives like next-token prediction. Fine-tuning adapts the pre-trained model on a smaller labeled dataset for a specific task. RLHF (Reinforcement Learning from Human Feedback) further aligns the model using human preferences as a reward signal, improving helpfulness and safety.

---

**Question**: Compare BPE, WordPiece, and SentencePiece tokenization.

**Answer**: BPE (Byte-Pair Encoding) iteratively merges the most frequent byte pairs, used in GPT models. WordPiece, used in BERT, merges pairs that maximize the likelihood of the training data. SentencePiece treats the input as a raw byte stream and can apply BPE or Unigram LM tokenization directly without pre-tokenization, making it language-agnostic and used in LLaMA.

```python
# BPE example using tiktoken (GPT-4 tokenizer)
import tiktoken
enc = tiktoken.get_encoding("cl100k_base")
tokens = enc.encode("Hello, world!")
print([enc.decode([t]) for t in tokens])
# Output: ['Hello', ',', ' world', '!']
```

---

**Question**: How does the context window work and what is sliding window attention?

**Answer**: The context window defines the maximum number of tokens the model can attend to at once. Sliding window attention limits each token to attend only to a fixed-size window of neighboring tokens, reducing O(n²) complexity to O(n·k). This is used in models like Mistral and Longformer to handle longer sequences efficiently.

---

**Question**: What is KV-cache optimization and how does it improve inference?

**Answer**: KV-cache stores the Key and Value tensors from previous decoding steps so they don't need to be recomputed at each timestep. This reduces redundant computation from O(n³) to O(n²) per step during autoregressive generation. It trades memory for speed and is essential for low-latency LLM inference.

```python
# Simplified KV-cache logic during decoding
def generate_with_kv_cache(model, input_ids, max_len):
    kv_cache = None
    for step in range(max_len):
        logits, kv_cache = model(input_ids, kv_cache=kv_cache)
        next_token = logits[:, -1, :].argmax(dim=-1)
        input_ids = next_token.unsqueeze(0)
```

---

**Question**: Explain zero-shot, few-shot, and chain-of-thought prompting.

**Answer**: Zero-shot prompting asks the model to perform a task without examples. Few-shot prompting provides a few input-output examples in the prompt to guide the model. Chain-of-thought (CoT) prompting instructs the model to reason step-by-step before producing the final answer, significantly improving performance on arithmetic and logical reasoning tasks.

```python
# Chain-of-thought prompt example
prompt = """Q: Roger has 5 tennis balls. He buys 2 more cans of 3 balls each. How many balls does he have?
A: Roger starts with 5 balls. 2 cans of 3 balls each is 6 balls. 5 + 6 = 11. The answer is 11.

Q: The cafeteria had 23 apples. They used 20 for lunch and bought 6 more. How many apples do they have?
A:"""
```

---

**Question**: Describe the RAG (Retrieval-Augmented Generation) architecture.

**Answer**: RAG combines a retrieval system with a generative model. Given a query, the retriever fetches relevant documents from a knowledge base, and the generator conditions on both the query and retrieved context to produce an answer. This grounds generation in factual knowledge, reduces hallucination, and enables updating knowledge without retraining.

```python
# Simplified RAG pipeline
def rag_pipeline(query, retriever, generator):
    docs = retriever.retrieve(query, k=3)
    context = "\n".join([d.text for d in docs])
    prompt = f"Context: {context}\n\nQuestion: {query}\nAnswer:"
    return generator.generate(prompt)
```

---

**Question**: What chunking strategies are used in RAG?

**Answer**: Fixed-size chunking splits documents into equal token windows, often with overlap to avoid losing context at boundaries. Semantic chunking splits at natural boundaries like paragraphs or sentences using embedding similarity. Recursive chunking applies multiple splitting strategies hierarchically (e.g., first by section, then by paragraph). Agentic chunking uses an LLM to identify logical breakpoints.

---

**Question**: How do embedding models and vector databases work together in RAG?

**Answer**: Embedding models convert text into dense vector representations (e.g., 768 or 1536 dimensions) that capture semantic meaning. Vector databases (e.g., Pinecone, Weaviate, Qdrant) index these embeddings using approximate nearest neighbor (ANN) algorithms like HNSW or IVF for efficient similarity search. At query time, the query is embedded and the database returns the most semantically similar chunks.

---

**Question**: Compare semantic search with keyword search.

**Answer**: Keyword search (BM25) matches exact terms using inverted indices with TF-IDF weighting, which is fast but fails on synonyms and paraphrases. Semantic search uses dense embeddings to capture meaning beyond exact word matches, handling synonyms and conceptual similarity. Hybrid search combines both by fusing BM25 scores with embedding similarity scores for optimal retrieval.

---

**Question**: Contrast full fine-tuning, LoRA, and QLoRA.

**Answer**: Full fine-tuning updates all model parameters, requiring significant compute and memory (e.g., 4× the model size in VRAM for AdamW). LoRA (Low-Rank Adaptation) injects trainable low-rank matrices into attention layers, updating only ~0.1-1% of parameters while freezing the base model. QLoRA combines LoRA with 4-bit quantization of the base model, enabling fine-tuning of 65B+ models on a single GPU.

```python
from peft import LoraConfig, get_peft_model

lora_config = LoraConfig(
    r=8,
    lora_alpha=32,
    target_modules=["q_proj", "v_proj"],
    lora_dropout=0.1,
)
model = get_peft_model(base_model, lora_config)
# Only LoRA parameters are trainable
```

---

**Question**: What is PEFT and what techniques fall under it?

**Answer**: PEFT (Parameter-Efficient Fine-Tuning) adapts large models by updating only a small fraction of parameters while freezing the rest. Techniques include LoRA / QLoRA (low-rank decomposition of weight updates), AdaLoRA (adaptive rank allocation), Prefix Tuning (learns virtual tokens prepended to input), and Prompt Tuning (learns soft prompts). PEFT reduces memory footprint and storage overhead per task.

---

**Question**: Compare INT8, FP4, GPTQ, and AWQ quantization methods.

**Answer**: INT8 quantization maps weights to 8-bit integers using symmetric or asymmetric scaling, losing minimal accuracy. FP4 uses 4-bit floating-point storage for higher precision near zero but runs on limited hardware. GPTQ performs one-shot weight quantization by optimizing based on the Hessian matrix, achieving high-quality 4-bit and 3-bit compression. AWQ (Activation-aware Weight Quantization) protects salient weight channels identified by activation magnitudes, outperforming GPTQ at extreme compression levels.

```python
# Loading a quantized model with bitsandbytes
from transformers import AutoModelForCausalLM, BitsAndBytesConfig

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.bfloat16
)
model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-2-7b-hf",
    quantization_config=bnb_config
)
```

---

**Question**: Explain temperature, top-k, and top-p sampling.

**Answer**: Temperature controls the sharpness of the output probability distribution — lower values (e.g., 0.1) make the model more deterministic, higher values (e.g., 1.5) increase randomness. Top-k sampling restricts output to the k most likely tokens before sampling. Top-p (nucleus) sampling selects the smallest set of tokens whose cumulative probability exceeds p, dynamically adjusting the candidate pool.

```python
def sample_with_temperature(logits, temperature=1.0, top_k=50, top_p=0.9):
    scaled = logits / temperature
    # Top-k filtering
    if top_k > 0:
        indices_to_remove = scaled < torch.topk(scaled, top_k)[0][..., -1, None]
        scaled[indices_to_remove] = float('-inf')
    # Top-p filtering
    probs = torch.softmax(scaled, dim=-1)
    sorted_probs, sorted_indices = torch.sort(probs, descending=True)
    cumulative_probs = torch.cumsum(sorted_probs, dim=-1)
    mask = cumulative_probs > top_p
    mask[..., 1:] = mask[..., :-1].clone()
    mask[..., 0] = False
    sorted_probs[mask] = 0.0
    probs.scatter_(-1, sorted_indices, sorted_probs)
    return torch.multinomial(probs, num_samples=1)
```

---

**Question**: Differentiate beam search and greedy decoding.

**Answer**: Greedy decoding selects the token with the highest probability at each step, which is fast but can miss better sequences due to local optima. Beam search maintains k candidate sequences (beams) at each step, scoring them by cumulative log-probability, and selects the best overall sequence at the end. Beam search produces higher-quality outputs for tasks like translation but is slower and more computationally expensive.

---

**Question**: What causes hallucination in LLMs and how can it be mitigated?

**Answer**: Hallucination arises from the model's auto-regressive nature filling plausible-sounding tokens, training data noise, or lack of factual grounding during generation. Mitigations include RAG (grounding in retrieved documents), prompt engineering (e.g., "only answer if you know"), controlled decoding (e.g., DoLa), fine-tuning with preference data, and confidence calibration with rejection sampling.

---

**Question**: Describe guardrails and content filtering in LLM applications.

**Answer**: Input guardrails detect harmful, toxic, or off-topic user queries before they reach the LLM using classifiers or regex patterns. Output guardrails validate the model's response for safety, PII leakage, and factual consistency before returning it to the user. Tools like NeMo Guardrails, Guardrails AI, and Azure AI Content Safety provide pre-built and customizable rail policies.

---

**Question**: Compare BLEU, ROUGE, METEOR, and perplexity as evaluation metrics.

**Answer**: Perplexity measures how well the model predicts a sequence — lower is better for language modeling. BLEU computes n-gram precision between generated and reference text, commonly used in translation. ROUGE measures n-gram recall, primarily for summarization. METEOR aligns unigrams and accounts for synonyms and stemming, correlating better with human judgment than BLEU alone.

```python
from evaluate import load
bleu = load("bleu")
results = bleu.compute(
    predictions=["the cat sat on the mat"],
    references=[["the cat is on the mat"]]
)
print(results["bleu"])  # ~0.66
```

---

**Question**: What is LLM-as-judge evaluation?

**Answer**: LLM-as-judge uses a strong LLM (e.g., GPT-4, Claude 3) to evaluate outputs of another model by providing scoring rubrics or pairwise comparisons. It assesses criteria like helpfulness, harmlessness, and instruction-following that are hard to capture with n-gram metrics. Calibration and positional bias should be mitigated by swapping response order and using structured scoring templates.

```python
judge_prompt = """You are an expert evaluator. Rate the following response on a scale of 1-5 for accuracy, conciseness, and safety.

Query: {query}
Response: {response}

Output format: "Score: {score}\nReasoning: {reasoning}" """
```

---

**Question**: What is instruction tuning and how does it differ from pre-training?

**Answer**: Instruction tuning fine-tunes a pre-trained LM on a dataset of instruction-response pairs (e.g., "Summarize this article: ..." → summary). It teaches the model to follow user instructions in a conversational format, unlike pre-training which only learns token probabilities. Models like InstructGPT, LLaMA-2-Chat, and Mistral-Instruct are built via instruction tuning.

---

**Question**: Explain the RLHF training pipeline.

**Answer**: RLHF has three stages: (1) supervised fine-tuning (SFT) on high-quality demonstrations, (2) training a reward model on human preference comparisons between model outputs, and (3) fine-tuning the SFT model using Proximal Policy Optimization (PPO) to maximize the reward while staying close to the SFT model via a KL penalty. This aligns the model with human values and preferences.

---

**Question**: How does DPO differ from RLHF?

**Answer**: DPO (Direct Preference Optimization) eliminates the need for a separate reward model and PPO training. It directly optimizes the policy on preference pairs using a binary cross-entropy loss derived from the Bradley-Terry preference model. DPO is simpler, more stable, and computationally cheaper than RLHF, while achieving comparable or better alignment results.

```python
# Simplified DPO loss
def dpo_loss(policy_chosen_logps, policy_rejected_logps,
             ref_chosen_logps, ref_rejected_logps, beta=0.1):
    log_ratio = (policy_chosen_logps - ref_chosen_logps) - \
                (policy_rejected_logps - ref_rejected_logps)
    loss = -torch.log(torch.sigmoid(beta * log_ratio)).mean()
    return loss
```

---

**Question**: What is model distillation in the context of LLMs?

**Answer**: Model distillation trains a smaller "student" model to mimic the behavior of a larger "teacher" model, typically by minimizing the KL divergence between their output distributions. It reduces model size and inference cost while retaining much of the teacher's capability. Examples include DistilBERT (60% speed, 97% performance of BERT) and task-specific distillation of GPT-4 into smaller models.

---

**Question**: Explain Mixture of Experts (MoE) architecture.

**Answer**: MoE replaces dense feed-forward layers with multiple "expert" sub-networks and a sparse gating mechanism that routes each token to only the top-k experts. This increases model capacity without proportional compute cost. Mixtral 8x7B uses 8 experts with top-2 routing, effectively having 47B parameters but only ~13B active per token.

```python
class SparseMoE(nn.Module):
    def __init__(self, d_model, num_experts=8, top_k=2):
        super().__init__()
        self.gate = nn.Linear(d_model, num_experts)
        self.experts = nn.ModuleList([
            nn.Sequential(nn.Linear(d_model, d_model * 4), nn.GELU(), nn.Linear(d_model * 4, d_model))
            for _ in range(num_experts)
        ])
        self.top_k = top_k

    def forward(self, x):
        gates = self.gate(x).softmax(dim=-1)
        top_k_vals, top_k_idx = torch.topk(gates, self.top_k, dim=-1)
        top_k_vals = top_k_vals / top_k_vals.sum(dim=-1, keepdim=True)
        out = torch.zeros_like(x)
        for i, expert in enumerate(self.experts):
            mask = (top_k_idx == i).any(dim=-1)
            if mask.any():
                out[mask] += top_k_vals[mask][:, (top_k_idx[mask] == i).float().argmax(dim=-1), None] * expert(x[mask])
        return out
```

---

**Question**: How do multi-modal models like GPT-4V and LLaVA work?

**Answer**: Multi-modal models project non-text modalities into the LLM's embedding space using a connector (e.g., a Q-Former or linear projection). An image encoder (e.g., ViT) generates visual features, and the connector maps them to token embeddings the LLM can attend to alongside text tokens. The model is fine-tuned end-to-end on interleaved image-text data, enabling visual reasoning.

---

**Question**: Describe the Vision Transformer (ViT) architecture.

**Answer**: ViT splits an image into fixed-size patches (e.g., 16×16), linearly projects each patch into an embedding, adds positional encodings, and processes them through a standard Transformer encoder. Unlike CNNs, ViT has no built-in spatial inductive bias and relies on pre-training on large datasets (e.g., JFT-300M) to learn spatial relationships. It achieves state-of-the-art image classification when sufficiently pre-trained.

---

**Question**: How do diffusion models generate images?

**Answer**: Diffusion models learn to reverse a gradual noising process. During training, Gaussian noise is added to an image over T timesteps, and the model learns to predict the added noise. During generation, the model starts from pure noise and iteratively denoises it over T steps. Text conditioning is injected via cross-attention, enabling text-to-image generation (e.g., Stable Diffusion, DALL-E 3).

```python
# Simplified diffusion sampling step
@torch.no_grad()
def sample_step(model, x_t, t, text_embedding):
    predicted_noise = model(x_t, t, text_embedding)
    alpha_t = alphas[t]
    alpha_bar_t = alphas_cumprod[t]
    x_t_minus_1 = (1 / torch.sqrt(alpha_t)) * (
        x_t - (1 - alpha_t) / torch.sqrt(1 - alpha_bar_t) * predicted_noise
    )
    if t > 0:
        x_t_minus_1 += sigma_t * torch.randn_like(x_t)
    return x_t_minus_1
```

---

**Question**: What are the main components of the LangChain framework?

**Answer**: LangChain provides abstractions for LLM interactions: Models (wrappers for LLMs, chat models, embedding models), Prompts (templates and few-shot examples), Chains (composable sequences of calls), Memory (state persistence across interactions), Agents (autonomous decision-making with tool access), and Retrieval (document loaders, text splitters, vector stores, retrievers).

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.runnables import RunnablePassthrough

prompt = ChatPromptTemplate.from_template("Answer based on: {context}\nQuestion: {question}")
model = ChatOpenAI(model="gpt-4")
chain = {"context": retriever, "question": RunnablePassthrough()} | prompt | model
result = chain.invoke("What is RAG?")
```

---

**Question**: How does LlamaIndex differ from LangChain for data indexing?

**Answer**: LlamaIndex specializes in data indexing and retrieval, offering rich abstractions for ingesting, chunking, and indexing documents (e.g., Document, Node, Index) with modular retrieval strategies. LangChain is a broader framework for chaining LLM calls, tools, and agents. LlamaIndex excels at RAG pipelines with advanced indexing (summary index, tree index, keyword table index) and complex query engines.

---

**Question**: What are token limits and how can context compression help?

**Answer**: Token limits constrain how much text the model can process in a single request (e.g., 4K, 8K, 128K tokens). Context compression reduces the size of retrieved documents before feeding them to the LLM, using techniques like extractive compression (selecting relevant sentences), summarization, or learned "soft compression" tokens. This saves costs, reduces latency, and fits more information into the context window.

---

**Question**: Describe caching strategies for LLM applications.

**Answer**: Semantic caching stores LLM responses keyed by the embedding of the query, returning cached results for semantically similar queries. Prefix caching caches KV-cache states for common prompt prefixes, reducing TTFT (time-to-first-token). Exact-match caching uses the raw text as the key. For RAG, cache retrieved document chunks to avoid repeated vector database lookups.

```python
from functools import lru_cache
import numpy as np

class SemanticCache:
    def __init__(self, threshold=0.95):
        self.threshold = threshold
        self.cache = []  # (embedding, response)

    def get(self, query_emb):
        for emb, response in self.cache:
            if np.dot(query_emb, emb) / (np.linalg.norm(query_emb) * np.linalg.norm(emb) + 1e-10) > self.threshold:
                return response
        return None

    def set(self, query_emb, response):
        self.cache.append((query_emb, response))
```

---

**Question**: How does streaming work in LLM APIs using SSE?

**Answer**: Server-Sent Events (SSE) streams tokens one at a time over an HTTP connection as the model generates them. The client receives `data: {"token": "Hello"}` events and incrementally renders text for lower perceived latency. Streaming uses chunked transfer encoding and the `text/event-stream` content type.

```python
# Client-side streaming
import httpx

async def stream_response(prompt):
    async with httpx.AsyncClient() as client:
        async with client.stream("POST", "https://api.openai.com/v1/chat/completions",
            json={"model": "gpt-4", "messages": [{"role": "user", "content": prompt}], "stream": True}
        ) as response:
            async for line in response.aiter_lines():
                if line.startswith("data: ") and line != "data: [DONE]":
                    yield line[6:]
```

---

**Question**: Explain function calling / tool use in LLMs.

**Answer**: Function calling allows the LLM to request execution of external functions by outputting structured JSON arguments. The developer defines tool schemas (name, description, parameters), and the model decides when to invoke them. Results are fed back into the conversation, enabling the model to perform actions like API calls, database queries, or calculations.

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get current temperature for a city",
            "parameters": {
                "type": "object",
                "properties": {"city": {"type": "string"}},
                "required": ["city"]
            }
        }
    }
]
response = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "What's the weather in Tokyo?"}],
    tools=tools
)
# response.choices[0].message.tool_calls contains the function call
```

---

**Question**: How do you generate structured output (JSON mode) from LLMs?

**Answer**: JSON mode constrains the model to produce valid JSON by setting `response_format={"type": "json_object"}` (OpenAI) or using constrained decoding libraries like Outlines, LMQL, or Guidance. These libraries mask the logits to only allow tokens that conform to a specified JSON schema or Pydantic model, guaranteeing syntactically correct output.

```python
from pydantic import BaseModel
from openai import OpenAI

class Movie(BaseModel):
    title: str
    year: int
    rating: float

response = client.beta.chat.completions.parse(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Extract movie info: Inception, 2010, 8.8"}],
    response_format=Movie
)
movie = response.choices[0].message.parsed
```

---

**Question**: What strategies exist for rate limiting and cost optimization with LLM APIs?

**Answer**: Implement exponential backoff with jitter for retries on 429 errors. Cache semantically similar queries (semantic caching). Batch smaller requests into larger ones where possible. Use smaller/cheaper models for simple tasks and route complex ones to larger models (model routing). Implement token budgeting, prompt compression, and request queuing to stay within tier limits.

---

**Question**: What is prompt injection and how do you defend against it?

**Answer**: Prompt injection is an attack where a user's input overrides the system prompt or instructs the model to ignore safety guidelines. Defenses include strict input sanitization, using a "system" vs "user" role separation, instruction boundaries (e.g., delimiters), guardrail classifiers, and structured output parsing. Never include sensitive data in the system prompt if users can inject text.

---

**Question**: How do you detect and redact PII in LLM applications?

**Answer**: PII detection uses regex patterns, named entity recognition (NER) models, or dedicated libraries like Microsoft Presidio or Google DLP to identify emails, SSNs, credit card numbers, etc. Redaction replaces detected PII with placeholders (e.g., "[REDACTED]") before sending data to the LLM or to the user. Implement both input and output guardrails to prevent PII leakage.

```python
from presidio_analyzer import AnalyzerEngine
from presidio_anonymizer import AnonymizerEngine

analyzer = AnalyzerEngine()
anonymizer = AnonymizerEngine()

text = "Contact john@email.com for help"
results = analyzer.analyze(text=text, entities=["EMAIL_ADDRESS"], language="en")
anonymized = anonymizer.anonymize(text=text, analyzer_results=results)
# 'Contact <EMAIL_ADDRESS> for help'
```

---

**Question**: How do you A/B test LLM prompts in production?

**Answer**: Define prompt variants as versioned templates and route traffic using a deterministic split (e.g., by user ID hash). Collect metrics on both variants: output quality (LLM-as-judge scores), latency, token usage, and downstream task metrics (e.g., user satisfaction). Use statistical significance testing (e.g., chi-square, t-test) to determine the winner before rolling out fully.

---

**Question**: How do you version prompts and models?

**Answer**: Store prompts as code in a version-controlled repository with a structured template format (e.g., JSON or Jinja2). Use prompt registries (e.g., LangSmith Hub, MLflow) for collaboration. Track model versions via the model registry (e.g., Hugging Face Model Hub, MLflow Model Registry). Each deployment should log the exact prompt template ID and model version for reproducibility.

---

**Question**: What considerations go into curating a fine-tuning dataset?

**Answer**: Ensure data quality (no duplicates, factual accuracy), diversity (covers target distribution), and correct labeling (inter-annotator agreement). Balance between instruction types and difficulty levels. Include edge cases and adversarial examples. Remove PII and toxic content. Use a held-out validation set and measure per-task performance before and after fine-tuning.

---

**Question**: How is synthetic data generated for LLM training?

**Answer**: Synthetic data is generated by a strong model (e.g., GPT-4, Claude) given seed prompts or a taxonomy. Common methods include self-instruct (model generates its own training pairs), back-translation (round-trip translation for paraphrases), and evolution (mutating existing instructions). Generated data must be filtered for quality and diversity, often using a teacher model as a judge.

```python
def generate_synthetic_qa(topic, teacher_model="gpt-4", num_samples=10):
    prompt = f"""Generate {num_samples} question-answer pairs about {topic}.
Format each as: Q: <question>\nA: <answer>"""
    response = teacher_model.generate(prompt)
    return parse_qa_pairs(response)
```

---

**Question**: How do you measure retrieval quality in RAG?

**Answer**: Key metrics include Precision@k (proportion of relevant documents in top-k), Recall@k (proportion of all relevant documents retrieved), Mean Reciprocal Rank (MRR, inverse rank of the first relevant document), and Normalized Discounted Cumulative Gain (NDCG, which accounts for graded relevance). These are computed over a labeled test set of query-relevant document pairs.

---

**Question**: What is re-ranking and why is it used in RAG?

**Answer**: Re-ranking takes the top-N results from an efficient first-stage retriever (e.g., embedding similarity or BM25) and scores them with a more expensive but accurate cross-encoder model. This improves precision by considering full query-document interactions rather than just vector similarity. Re-rankers typically use models like Cohere Rerank, BGE-reranker, or MonoBERT.

```python
# Cross-encoder re-ranking
from sentence_transformers import CrossEncoder

reranker = CrossEncoder("cross-encoder/ms-marco-MiniLM-L-6-v2")
pairs = [[query, doc] for doc in retrieved_docs]
scores = reranker.predict(pairs)
reranked = [doc for _, doc in sorted(zip(scores, retrieved_docs), key=lambda x: x[0], reverse=True)]
```

---

**Question**: What is hybrid search (dense + sparse) and why use it?

**Answer**: Hybrid search combines dense retrieval (embedding similarity) and sparse retrieval (BM25 keyword matching) by fusing their scores, typically using Reciprocal Rank Fusion (RRF) or weighted summation. Dense search captures semantic similarity while sparse search ensures exact keyword matches are not missed. This is essential for domains with specialized terminology or where high recall is critical.

```python
def hybrid_search(query, dense_retriever, sparse_retriever, alpha=0.5):
    dense_scores = dense_retriever.retrieve(query, score_key="dense")
    sparse_scores = sparse_retriever.retrieve(query, score_key="sparse")
    combined = {}
    for doc_id in set(dense_scores) | set(sparse_scores):
        d = dense_scores.get(doc_id, 0)
        s = sparse_scores.get(doc_id, 0)
        combined[doc_id] = alpha * d + (1 - alpha) * s
    return sorted(combined.items(), key=lambda x: x[1], reverse=True)
```

---

**Question**: Explain multi-hop RAG.

**Answer**: Multi-hop RAG answers questions requiring information from multiple documents through iterative retrieval and reasoning. The system first retrieves documents for the initial query, extracts entities or missing information, formulates follow-up queries, retrieves additional documents, and repeats until sufficient context is gathered. This is essential for complex questions like "Which company founded by the inventor of the Post-it Note also produces adhesives?"

---

**Question**: What is Agentic RAG and how does it differ from standard RAG?

**Answer**: Agentic RAG adds an autonomous agent layer that decides when and what to retrieve, which tools to call, and how to synthesize information. Unlike standard RAG (single retrieval → generate), the agent can iteratively refine queries, choose between multiple data sources, call APIs, execute code, and combine results. Frameworks like LangChain Agents, AutoGPT, and CrewAI enable this pattern.

```python
class AgenticRAG:
    def answer(self, query):
        plan = self.llm.plan(f"Break down this question into sub-queries: {query}")
        context = []
        for step in plan.steps:
            if step.type == "retrieve":
                docs = self.retriever.retrieve(step.query)
                context.extend(docs)
            elif step.type == "compute":
                result = self.code_executor.run(step.code)
                context.append(f"Computed: {result}")
        return self.llm.generate(f"Query: {query}\nContext: {context}\nAnswer:")
```
