# OrchestrAI 

> Building a policy-governed AI agent for diagnosing and remediating failures in legacy enterprise systems.
---

# Problem
In enterprise environments, when a backend service goes down, someone has to:
1. **Notice it**
2. **Figure out why**
3. **Decide what to do**
4. **Fix it**

One traditional approach to automate this workflow relied on **rule-based diagnostic systems**, such as:

- Threshold-based alerts  
  *(e.g., if CPU > 90% for 5 minutes → restart service)*
  This worked well in **legacy monolithic systems**, but begins to break down as systems become:

 - distributed
 - event-driven
 - loosely coupled

---

## Limitations of Rule-Based Systems

### 1. Combinatorial Explosion
Modern systems introduce too many interacting variables:
- logs
- metrics
- retries
- upstream dependencies

Rules become:
- brittle
- hard to maintain
- incomplete

---

### 2. Context Fragmentation
Rule engines can only reason over:

> **the data they were explicitly wired to inspect**

But in real systems:
- logs live in one place
- metrics in another
- configs elsewhere

No single rule has full context.

---

# The Solution

> **An AI agent with an LLM "brain" for reasoning orchestration**

Instead of hardcoded rules:

- The system **collects evidence**
- The LLM **reasons across context**
- The agent **proposes actions**
- A policy layer **controls execution**

---

### Tech Stack
| Category           | Stack                           |
|--------------------|---------------------------------|
| Native Integration | Java Project Panama (FFM API)   |
| Agent Framework    | LangChain4j                     |
| Language           | Java / Spring Boot              |
| Inference Runtime  | ONNX Runtime GenAI              |
| Example model      | Quantized Phi-3 INT4 ONNX model |


# Architecture Overview
## Maven Multi-module Design
orchestrai-root

├── inference-runtime/ # pure inference (Panama + ONNX)

└── orchestrai-backend/ # agent reasoning + orchestration

### Key Principle

> **Inference and orchestration are decoupled at the module level, but run in the same process**

---

## Why This Architecture?

### Benefits

- Eliminates REST/network overhead
- Enables token-level streaming
- Keeps latency low
- Simplifies deployment (single JVM process)

---

### Trade-offs

- Native runtime failures can crash the JVM
- Less isolation compared to HTTP-based inference
- Requires careful memory management

---

# Design Decisions

---

## 1. ONNX Runtime + Project Panama

### Why?

### Eliminate Process Boundary

Instead of:

- **Java -> HTTP -> Model Server -> Response**
We use:

- **Java -> Panama -> NativeONNX Runtime**

---

### Lower Latency

- No serialization/deserialization
- No socket overhead
- No JSON parsing

This is critical for:

> **token-by-token streaming**

---

### Memory Control

- Direct access to off-heap memory
- Efficient handling of INT4 quantized models
- Fine-grained control of inference buffers

---

### Trade-offs

- Native memory is not GC-managed
- Memory leaks are your responsibility
- A native crash can bring down the entire JVM

---

## 2. LangChain4j Integration

LangChain4j is used for **agent orchestration**, not inference.

---

### State Management

> LLMs are stateless by default

LangChain4j provides:
- Chat memory
- Context persistence

This allows the agent to remember:

- what it already tried
- what failed
- what to try next

---

### Tool / Function Calling

This is the **core capability** of the system.

Example:

LLM Output → restart service

↓
LangChain4j → Java Method Call

↓

serviceRegistry.restart("payment-service")

---

### Structured Output Handling

LLM output is messy (text).

LangChain4j helps convert:

"Restart the domain"

into:

```java
restartDomain("domain1");
```
via structured extraction → POJOs.

Operational Signals (logs, metrics, endpoints)
↓

LLM Reasoning (ONNX Runtime)
↓

Structured Action Proposal
↓

Policy Validation
↓

Tool Execution
↓

Verification


## Developer Notes (Ongoing Learnings)
- **LLM output must always be treated as untrusted input**
- **Native memory ≠ JVM memory (must manage explicitly)**
- **Streaming tokens ≠ safe execution signals**
- **Policy layer is more important than model intelligence**


## Current Focus
- **Building inference runtime abstraction**
- **Implementing agent reasoning loop**
- **Designing policy validation layer**
- **Integrating tool execution safely**

### End Goal

A system that demonstrates:
- **Native systems integration (Panama + ONNX)**
- **Local LLM serving infrastructure**
- **Agent-based orchestration**
- **Safe AI-driven remediation**
- **Enterprise-grade design thinking**

### Next Steps
- **Implement agent loop (Observe → Reason → Act)** 
- **Add structured action parsing**
- **Integrate policy engine**
- **Build tool execution layer**
- **Add verification pipeline**
