# Problem
In enterprise environments, when a backend service goes down, someone has to:
1. **Notice it**
2. **Figure out why**
3. **Decide what to do**
4. **Fix it**

ONE way to automate that workflow relied on rule based diagnostics systems such as using regex-heavy log parsers and threshold-based alerts (e.g., if CPU > 90% for 5m, restart service).
This stack worked for years in most legacy enterprise monolithic systems. However, as systems got more distributed, the numerous failure modes becomes hard for humans to encode. A single incident can depend on GC pauses, network latency, DB pool exhaustion, e.t.c 
Another problem with rule-based heuristics is context. It can only reason over the data it was wired to inspect and these data is not necessarily in one place. 
**The Solution: LLM-driven orchestration**


### Tech Stack
| Category      | Stack                               |
|---------------|-------------------------------------|
| Integration   | Java 21 (Project Panama FFM API)    |
| Orchestration | LangChain4j                         |
| Integration   | Spring Boot 3                       |
| Runtime       | ONNX Runtime GenAI                  |
| Model         | Phi-3 INT4                          |
