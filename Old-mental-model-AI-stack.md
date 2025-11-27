# LLM-systems-stack (The way I think)
# Date 13th Nov, 2025
This is a repository of how I think about the "LLM Stuff" in 11 different layers. It contains practical LLM systems portfolio – experiments, code, and notes across 11 layers (transformers, RAG, agents, evals) from a product manager’s perspective.

## **Layer 1 – Hardware & Infra (the bottom of the stack)**

**What it is?**

The physical + cloud stuff that makes big models possible.

**Key concepts:**

- **GPUs / TPUs** – Specialized chips for matrix math.
- **Clusters** – Many GPUs tied together with fast interconnect.
- **Inference vs Training** – Training is heavy, long-running; inference is answering requests in real time.
- **Scaling** – Horizontal (more machines) vs vertical (bigger machines).

**PM-level understanding:**

- Training huge base models is something only big labs/clouds do.
- PM cares about **inference infrastructure**: This is where latency, reliability, and cost per request come from.
- You don’t manage GPUs, but you choose deployment shape (hosted API vs self-hosted, small vs big model).

---

## **Layer 2 – Data & Pretraining**

**What it is?:**

How we “fill the brain” before it ever sees your product.

**Key concepts:**

- **Pretraining corpus** – Web pages, code, books, etc.
- **Self-supervised learning** – Model learns to predict the next token from context, no manual labels.
- **Scale laws** – More data + parameters + compute → better performance (with diminishing returns).

**PM-level understanding:**

- Base models know *general* world knowledge up to their training cut-off.
- They *don’t* know your internal product docs, customer configs, or freshest info unless you add it later.

---

## **Layer 3 – Model Architecture (Transformers themselves)**

This is the “brain design.”

**Key concepts:**

- **Transformer** – The architecture behind modern LLMs.
- **Tokens** – Units of text; pricing, latency, and context all measured in tokens.
- **Attention / Self-attention** – Mechanism to “focus” on the most relevant previous tokens.
- **Context window** – Max tokens the model can see in a single request (8k, 32k, 128k, etc.).
- **Parameters** – Model size (billions of weights).

**PM-level understanding:**

- This explains context limits and why long prompts are expensive. You think in tokens, context sizes, and “what really needs to be in the prompt”.
- More parameters/context generally = smarter but slower/more expensive.
- The **context window** is a hard constraint → why you need RAG, summarization, etc.

---

## **Layer 4 – Base Model Training & Alignment**

This is taking the raw transformer and making it not-insane and more “helpful.”

**Key concepts:**

- **Pretrained base model** – Raw model after next-token training.
- **Instruction tuning** – Train on “instruction → response” pairs (“Write an email…”, “Explain…”).
- **RLHF / RLAIF** – “Reinforcement Learning from Human/AI Feedback” to make the model more helpful, safe, less toxic.
- **Guardrails during training** – Filtering bad data, safety objectives.

**PM-level understanding:**

- PM's don’t need to run this, but:
    - Different vendors/models behave differently because of alignment.
    - Models differ in **helpfulness**, **safety**, **coding skill**, etc. because of how they were aligned.
    - Your choice of model family matters for UX and risk (e.g. code-heavy vs general chat vs small on-device).

---

## **Layer 5 – Adaptation to Your Use Case (Prompts & Fine-tuning)**

This is where your product’s needs come in. This is exactly where you specialize a general model for your product.

**Key concepts:**

- **Prompting**
    - System prompts (persona, rules).
    - Few-shot examples.
    - Structured output (JSON, tables).
- **Fine-tuning**
    - Training on *your* examples to specialize behavior.
    - **LoRA / adapters** – Memory-efficient ways to fine-tune.
    - **Domain adaptation** vs **task-specific tuning**.
- **Guardrails at generation time**
    - Refusal policies, style constraints, etc.

**PM-level understanding:**

- **Prompting** = first, cheapest lever to make it do what you want.
- **Fine-tuning** = when you have lots of labeled examples and need consistency in narrow tasks.
- Big product decision: “Can we solve this with prompts + RAG, or do we invest in fine-tuning?”

---

## **Layer 6 – Knowledge & Tools (RAG, Search, APIs, DBs)**

This is the “LLM + your data + your systems” layer.

### **6a. Knowledge / Retrieval (RAG)**

**Key concepts:**

- **Embeddings** – Vector representation of text for similarity search.
- **Vector database** – Stores embeddings + metadata and returns “similar” chunks.
- **Chunking** – Splitting docs into pieces that fit in context.
- **RAG (Retrieval-Augmented Generation)** – Retrieve relevant chunks and feed them in with the prompt.
- **Hybrid search** – Combine keyword and vector search.

**PM-level understanding:**

- RAG is the default way to connect the model to your private and frequently-changing data.
- This is where you answer “How does the AI know our stuff?” and “How does it do things (not just talk)?”. You design data sources, retrieval strategy, and tool specs.
- You design: what data to ingest, how to chunk, what metadata, what sources to show to users.


### **6b. Tools & External Systems**

**Key concepts:**

- **Tool use / function calling** – Model decides when to call APIs (e.g. “get_user_invoice(id)”).
- **Structured tools** – Pre-defined tool schemas with arguments (JSON).
- **Databases & search indices** – SQL, analytics DBs, code search, etc.
- **Non-LLM models** – Recommenders, classifiers, scoring models.

**PM-level understanding:**

- The LLM becomes a **controller**: it chooses when to call tools, and then we feed results back into it.
- This is how you go from “chatbot” to “do stuff for me” (actions).

---

## **Layer 7 – Orchestration / Workflows**

Above single calls, you get **flows** and **pipelines**.

**Key concepts:**

- **Chains / Pipelines** – Multi-step flows:
    - e.g. “Classify → retrieve docs → call LLM → summarize → format JSON”.
- **State machines / workflow engines** – Representing branching logic & retries.
- **Routing** – Decide which model/flow to use based on input.
- **Caching** – Store results for repeated queries.
- **Fallbacks / multi-model** – Try a small model, fallback to big one on failure.

**PM-level understanding:**

- When you describe “the feature flow,” engineers will implement it as a chain/workflow.
- You decide: where humans step in, where to log decisions, where to add approvals.
- In other words, this mirrors your user flow diagrams. Your product spec (“first classify, then fetch docs, then draft answer, then summarize”) becomes a machine-readable workflow

---

## **Layer 8 – Agents (Single-Agent Systems)**

Now we’re at the “agent” buzzword you hear everywhere.

**Working definition (practical):**

An **agent** is an LLM loop that:

1. Has a **goal** (“Plan my 4-day Mexico trip”).
2. Can **take actions** using tools (search, DB, APIs).
3. Maintains some kind of **memory/state** across multiple steps.
4. **Self-reflects**: checks its own work, revises, or plans next steps.

**Key concepts:**

- **Planning vs acting** – The model decides steps (plan) and then executes them.
- **ReAct pattern** – “Reason + Act”: think step-by-step, then call tools.
- **Memory**
    - Short-term: conversation history, scratchpad.
    - Long-term: separate store of facts about user/tasks.
- **Critic / self-check** – Agent reviews its own outputs or asks a second model to critique.
- **Task decomposition** – Break big goals into smaller sub-tasks.

**PM-level understanding:**

- An “agent” isn’t magic – it’s just an LLM + tools + loop + state.
- This is “one smart worker” in your product. You define what it’s allowed to do, what tools it has, and when it should escalate to a human.
- You define:
    - What tools it has.
    - What tasks it’s allowed to do.
    - How safe it needs to be (what it *must not* do).
    - When it should escalate to a human.

Example:

A “billing support agent” that:

- Looks up your account,
- Checks invoices,
- Creates a credit note via API,
- Writes you an explanation—all via tools orchestrated around an LLM.

---

## **Layer 9 – Multi-Agent Systems & Orchestration**

Now we have **multiple agents** with different roles working together.

**What it is:**

Instead of one big “do-everything agent,” we have specialized ones, e.g.:

- **Planner** – Breaks a big problem into subtasks.
- **Researcher** – Good at retrieval + reading.
- **Coder** – Good at writing and debugging code.
- **Reviewer** – Checks quality and consistency.
- **Coordinator / Supervisor** – Oversees the whole process.

**Key concepts & patterns:**

- **Role-based agents** – Each has a clear job, possibly different tools.
- **Messaging/communication** – Agents send messages to each other (like a Slack for bots).
- **Orchestrator** – A controller that:
    - Assigns tasks to agents,
    - Routes information,
    - Decides when you’re “done.”
- **Debate / ensemble** patterns – Two agents argue or produce alternatives; another picks the best.
- **Task marketplaces** – Think “job board for agents”: tasks posted, agents bid/claim.

**PM-level understanding:**

- Multi-agent systems shine when:
    - Tasks are complex and decomposable,
    - Specialization helps (code vs copy vs analysis),
    - You want redundancy/quality via multiple opinions.
- They also add complexity: latency, logging, debugging, costs.
- As PM, you’d decide: “Do we really need many agents, or is one carefully-designed agent + tools enough?”
- You only go here when the task is complex enough to benefit from specialization and redundancy. You weigh added complexity vs gains in quality and robustness.

---

## **Layer 10 – Observability, Evaluation, Safety, Governance**

Across all layers, you need **control & measurement**.

**Key concepts:**

- **Offline evals** – Test sets with “gold” answers; measure accuracy, relevance, toxicity.
- **Online metrics** – Conversion, deflection, time saved, CSAT, retention.
- **Tracing** – Logging every step: which tools, which prompts, which agents.
- **Guardrails**
    - Input validation: block certain prompts.
    - Output filtering: PII, toxicity, hallucination checks.
    - Policy enforcement: don’t perform certain actions without human approval.
- **Governance**
    - Who can ship prompts or tools?
    - Approval & review of changes, versioning.
    - Data residency / privacy rules.

**PM-level understanding:**

- This is where you define **acceptance criteria** for AI behavior.
- You should own the eval plan: “What does ‘good enough’ look like? How do we know if we broke something?”

---

## **Layer 11 – Product Layer (UX, Jobs-To-Be-Done, Business)**

Finally, the part that’s *most* your world.

**Key concepts:**

- **Use case & JTBD** – e.g. “reduce time to answer tickets by 40%,” “let PMs auto-draft specs.”
- **Interaction patterns** – Chat, search, autocomplete, command palette, background agents.
- **Human-in-the-loop** – Review steps, approvals, “click to apply changes.”
- **Feedback loops** – Thumbs up/down, “report issue,” fine-tuning data.
- **Pricing & packaging** – Which features are where, usage-based pricing, cost controls.

**PM-level understanding:**

- You translate user needs into:
    - Which layers we need,
    - What constraints exist (privacy, latency, cost),
    - How we measure impact.
