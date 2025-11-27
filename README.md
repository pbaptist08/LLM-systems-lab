# LLM Stack – A Practical 5-Layer Map for Product Managers

This document describes a **5-layer mental model** for modern LLM systems and a **learning flow for PMs** who want enough depth to design and ship AI features (not become researchers).

The five layers are:

| # | Layer Name           | What it covers                                                                                      | Sub-layers (from original 11-layer view)                              |
|---|----------------------|-----------------------------------------------------------------------------------------------------|------------------------------------------------------------------------|
| 1 | **Infra**            | Physical and cloud infrastructure needed to train and run LLMs                                      | Hardware & Infra (1)                                                  |
| 2 | **Models**           | How LLMs are built and aligned before being used in products                                        | Data & Pretraining (2), Transformer Architecture (3), Training & Alignment (4) |
| 3 | **Data & Knowledge** | How models are adapted to a product and connected to data (prompts, RAG, tools, APIs, DBs)          | Adaptation via Prompts & Fine-tuning (5), Knowledge & Tools / RAG (6) |
| 4 | **Orchestration**    | How all components run together in production, including workflows, agents, evals, and safety       | Workflows/Chains (7), Single & Multi-Agent Systems (8, 9), Evals & Safety & Governance (10) |
| 5 | **App / Product**    | The actual product: UX, JTBD, interaction patterns, pricing, adoption, and feedback loops           | Product Layer – UX & Business (11)                                    |

The original “11 layers” are folded into these five, but the intent and detail are preserved.

---

## How a PM Should Learn This Stack (High-Level Flow)

A product manager does **not** need equal depth in every layer.

A sensible learning sequence:

1. **Start with Models (Layer 2)**  
   - Understand transformers, tokens, context windows, and why different models behave differently.  
   - This gives the mental foundation for everything else.

2. **Move quickly to Data & Knowledge (Layer 3)**  
   - This is the core of practical PM work: prompting, RAG, embeddings, tools, connecting the model to product data.  
   - Hands-on project work should primarily live here.

3. **In parallel, think about App / Product (Layer 5)**  
   - Map AI capabilities to real user jobs, UX patterns, and metrics.  
   - Anchor learning in one or two concrete example features or side projects.

4. **Then add Orchestration (Layer 4)**  
   - Understand how chains and agents fit together, and how evaluation and safety wrap around them.  
   - Use simple flows and a basic eval plan rather than over-engineering.

5. **Keep Infra (Layer 1) as background context**  
   - Enough to reason about latency, cost, and limits—no need to go deep into GPU details.

The rest of this README describes each layer and suggests **what depth a PM needs** and **what to practice or document** at that layer.

---

## Layer 1 – Infra

**Question answered:**  
> What physical and cloud infrastructure is needed to train and run LLMs, and why does it matter for product decisions?

### What this layer includes

- GPUs / TPUs and other accelerator hardware  
- Clusters and high-speed interconnects  
- Training vs inference workloads  
- Horizontal vs vertical scaling  
- Latency, throughput, rate limits  
- Cost per token / per request  
- Reliability and SLOs

### How deep a PM needs to go

- **Conceptual only.**  
  - Understand that large models need specialized hardware and that **inference is not free**.  
  - Recognize that model choice + context size = latency + cost trade-offs.

### Practical PM questions at this layer

- How fast does the feature need to feel? (sub-second, 2–3 seconds, 10+ seconds is acceptable?)  
- How often will it be called? (per user session, per click, in the background?)  
- Is it okay to call a “big, expensive” model all the time, or should a smaller/cheaper model be used for most cases with fallbacks?

**Suggested learning artifact**

- A short note (1–2 pages) explaining:
  - Training vs inference  
  - Cost & latency trade-offs  
  - How this affects design of one example feature (e.g., why a small model might be used for “instant suggestions” and a bigger model for “full analysis”).

---

## Layer 2 – Models

**Question answered:**  
> What actually is an LLM under the hood, and why do different models behave differently?

This is the **conceptual backbone**. Once this is clear, later layers (RAG, prompting, context limits) make intuitive sense.

### 2.1 Data & Pretraining

- Pretraining corpus:
  - Web pages, code, books, documentation, etc.  
- Self-supervised learning:
  - Model learns to predict the next token from previous tokens.  
- Scale laws:
  - More data, parameters, and compute generally improve performance, with diminishing returns.  
- Training cut-off:
  - Models only know world events and content up to a certain date.

**PM depth:**  
Enough to explain that:

- A base model has **general world knowledge**, not product-specific knowledge.  
- It will not know about **recent policies or internal docs** unless given context via RAG or fine-tuning.

---

### 2.2 Transformer Architecture

- Tokens & tokenization  
- Self-attention:
  - Mechanism for deciding which previous tokens to “focus on”.  
- Context window:
  - Fixed limit on how many tokens the model can see at once (e.g., 8k, 32k, 128k).  
- Parameters:
  - Number of weights (billions), affecting capacity, quality, and cost.

**PM depth:**

- Be able to describe, in simple language:
  - That input text is broken into tokens.  
  - The model looks back over previous tokens using attention.  
  - The context window is a hard cap: not all data can be stuffed into one prompt.  
- Use this to reason about:
  - Why long inputs need summarization or retrieval.  
  - Why context engineering and RAG exist at all.

---

### 2.3 Training & Alignment

- Base model:
  - Raw model after pretraining, not user-friendly.  
- Instruction tuning:
  - Training on instruction → response examples (“Explain X…”, “Write an email to…”).  
- RLHF / RLAIF:
  - Reinforcement Learning from Human/AI Feedback to improve helpfulness, safety, tone.  
- Different model families:
  - General chat, code-focused, smaller on-device, domain-specialized.

**PM depth:**

- Be able to answer:
  - Why Model A is more “chatty” and Model B is more “code-focused”.  
  - Why different vendors’ models feel different (alignment policies, training data, objectives).  
- Use this when selecting a model for a feature (e.g., “support assistant vs code assistant”).

**Suggested learning artifacts**

- A concise markdown note:
  - “What is a transformer?” in 8–10 steps.  
  - What “tokens”, “context window”, and “parameters” mean for product decisions.  
- A small table comparing a few model families (OpenAI, Anthropic, Gemini, etc.) and what they’re good at.

---

## Layer 3 – Data & Knowledge  
*(Prompts, RAG, Tools)*

**Question answered:**  
> How is a general LLM adapted to a specific product, and how does it see the product’s data?

This is the **core practical layer** for product managers building LLM features.

### 3.1 Adaptation via Prompts & Fine-Tuning

This is where **“prompt engineering”** and “context engineering” live.

#### Concepts

- System prompts:
  - Instructions that set role, behavior, and constraints (“You are a cautious recipe assistant…”).  
- User prompts:
  - The actual user input or structured request built from UI.  
- Few-shot prompting:
  - Adding examples (“When the user says X, respond like Y”).  
- Structured output:
  - JSON, tables, or fixed templates that downstream code can parse reliably.  
- Fine-tuning:
  - Training on labeled examples to specialize behavior for narrow tasks.  
- Lightweight tuning:
  - LoRA, adapters, parameter-efficient fine-tuning techniques.

#### PM depth

- Understand that **prompting is the first lever**:
  - Many behaviors can be shaped without fine-tuning if prompts are carefully designed.  
- Know when fine-tuning is worth it:
  - Narrow tasks, high volume, requirement for consistent style or decisions, enough labeled examples.

#### Suggested practice

- Design at least one **system prompt** and iterate with:
  - Clear role, constraints, and output format.  
  - Document test cases (good, edge, fail).  
- Write a short note on when a team might move from “prompt only” to fine-tuning.

---

### 3.2 RAG, Embeddings, and Tools (Connecting to Data & APIs)

This is where the LLM is grounded in **product knowledge**.

#### RAG (Retrieval-Augmented Generation)

- Embeddings:
  - Numerical vectors representing text; similar meaning = close vectors.  
- Vector databases:
  - Specialized or extended DBs (e.g., Supabase + pgvector, Pinecone) that can store embeddings and do similarity search.  
- Chunking:
  - Splitting documents into smaller pieces with overlaps so retrieval is more targeted.  
- Classic RAG loop:
  1. Index content → store as chunks + embeddings.  
  2. At query time, embed the query.  
  3. Retrieve top-k similar chunks.  
  4. Inject them into a `Context:` section in the prompt.  
  5. Ask the LLM to answer using that context.

#### Tools & APIs

- Function calling:
  - Giving the model structured tools it can request, e.g. `get_user_profile(id)`, `fetch_orders(userId)`.  
- External systems:
  - SQL databases, analytics stores, search services, other ML models.  
- Orchestration around tools:
  - Passing arguments, validating, handling errors.

#### PM depth

- Be able to describe **why RAG is used**:
  - To access **private, up-to-date, or opinionated** knowledge that is not in the base model.  
- Understand design decisions:
  - What to store in the KB (policies, patterns, FAQs, logs).  
  - How to chunk and tag content.  
  - When to combine RAG with tools (e.g., “retrieve docs” + “query DB”).  
- Be ready to discuss:
  - How RAG improves **groundedness**, **consistency**, and **explainability** (“here are the sources used”).

#### Suggested practice

- Build or at least design a small RAG-backed feature:
  - Curated KB in a DB (e.g., Supabase),  
  - Embeddings + vector search,  
  - Prompt that includes a `Context:` section from retrieved chunks,  
  - UI that exposes “Why this answer?” (patterns / sources used).  
- Document the RAG architecture in a short diagram or markdown file.

---

## Layer 4 – Orchestration  
*(Workflows, Agents, Evals, Safety)*

**Question answered:**  
> How do all the pieces (prompts, RAG, tools, models) run together in production, and how is the system evaluated and governed?

### 4.1 Workflows / Chains

- Chains / pipelines:
  - Multi-step flows like “classify → retrieve → call LLM → post-process → save → respond”.  
- Routing:
  - Choosing which model or chain to use based on input type or risk.  
- Caching:
  - Reusing results for repeated queries.  
- Fallbacks:
  - Using simpler or smaller models first, falling back to more capable models when needed.

**PM depth:**

- Represent a feature as a simple flow:
  - Inputs → steps → outputs.  
- Understand where humans or approvals fit.  
- Know how additional steps impact latency, reliability, and cost.

---

### 4.2 Agents (Single & Multi-Agent)

- Single-agent systems:
  - One LLM loop that:
    - Has a goal,  
    - Uses tools,  
    - Maintains state,  
    - Plans and executes multiple steps.  
- Multi-agent systems:
  - Multiple agents with different roles:
    - Planner, researcher, coder, reviewer, supervisor.  
  - Agents passing messages and tasks between each other.  
- Debate / ensemble:
  - Multiple agents generate alternatives; another selects or merges.

**PM depth:**

- Recognize that an “agent” is not magic; it is an LLM + tools + a loop.  
- Decide when a **simple chain** is enough vs when an agent is justified:
  - Complex, decomposable tasks  
  - Need for iterative refinement or multiple tool calls  
  - Need for multiple “opinions” on critical tasks

For many MVPs, **chains + RAG + a well-designed prompt** are sufficient; agents can be future extensions.

---

### 4.3 Observability, Evaluation, Safety, Governance

- Offline evaluation:
  - Test sets of inputs with expected outputs or quality labels.  
- Online metrics:
  - Conversion, deflection, time saved, CSAT, error rates.  
- Tracing:
  - Logging prompts, retrieved context, tool calls, and responses for debugging.  
- Guardrails:
  - Input filters (block certain prompts),  
  - Output filters (PII, toxicity, hallucinations about restricted topics),  
  - Policy rules for certain actions.  
- Governance:
  - Who can change prompts, tools, and workflows; change management and approvals.

**PM depth:**

- Define “good enough” behavior for an AI feature.  
- Design a basic eval plan:
  - Test cases, success/failure criteria, manual review process.  
- Be able to explain how the system would be monitored in production and how regressions would be caught.

**Suggested practice**

- For any side project:
  - Create a small set of test cases and document what a “good” vs “bad” output looks like.  
  - Capture a few traces (prompt + context + response) to reason about failures.

---

## Layer 5 – App / Product

**Question answered:**  
> What is the actual product, who is it for, how does the AI show up in UX, and how is success measured?

### What this layer includes

- Use case & Jobs-To-Be-Done (JTBD)  
- Target personas and workflows  
- Interaction patterns:
  - Chat, forms, autocomplete, command palettes, background agents, suggestions.  
- Human-in-the-loop:
  - When the user must review or approve AI output.  
- Feedback mechanisms:
  - Ratings, flags, inline edits used as signals.  
- Pricing & packaging:
  - Free vs paid features, usage limits, AI add-ons.  
- Adoption & impact metrics:
  - Usage, retention, time saved, revenue impact, quality.

### PM depth

- Be able to define:

  - **Who** the AI feature serves (persona).  
  - **What job** it solves (JTBD).  
  - **Why AI** is appropriate (flexible reasoning, working with messy input, automation).  
  - **How** the interaction should feel (chatty, precise, background helper, etc.).  
  - **What metrics** define success (e.g., time saved, reduced manual effort, higher completion rates).

- Tie the underlying layers back to this:
  - Why prompting and RAG choices support the UX.  
  - Why orchestrations (chains/agents) are structured in a certain way.  
  - How evaluation and guardrails protect users and the business.

### Suggested practice

- For each AI project or side project:
  - Write a short “Product Notes” or mini-PRD including:
    - Problem and JTBD  
    - Target users  
    - AI’s role (assistant, generator, summarizer, decision helper, etc.)  
    - Chosen interaction pattern (form vs chat vs suggestions)  
    - High-level design of how prompts + RAG + tools serve that UX  
    - Success metrics and basic evaluation plan

---

## Recommended Learning Flow for PMs (Summary)

1. **Models (Layer 2)**  
   - Learn transformers, tokens, context windows, and model families.  
   - Output: short explainer notes + one comparison table of a few model options.

2. **Data & Knowledge (Layer 3)**  
   - Go deep on prompting, context engineering, RAG, embeddings, and tools.  
   - Output: a small working feature or side project (e.g., recipe assistant, support helper) with:  
     - A designed system prompt,  
     - A simple or moderate RAG setup,  
     - Notes on how the KB and prompt are structured.

3. **App / Product (Layer 5)**  
   - Wrap the feature in a clear product story: JTBD, UX, metrics.  
   - Output: a mini-PRD or product notes, plus a simple UI that demonstrates the feature.

4. **Orchestration (Layer 4)**  
   - Describe the end-to-end flow as a chain; decide where agents might fit in the future; design a basic eval plan.  
   - Output: a simple diagram / markdown of the flow and a small evaluation checklist.

5. **Infra (Layer 1)**  
   - Maintain a high-level understanding of cost and latency constraints and how they influence model selection and UX, without needing deep systems knowledge.

This 5-layer structure keeps the richness of the original 11-layer view but groups it into **manageable “drawers”** that align with how PMs actually work: from **product** down into **data/knowledge and models**, wrapped by **orchestration and infra constraints**.
