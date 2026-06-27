# Agentic AI – Interview Questions & Answers

**Question**: What is an AI Agent and what are its core components?

**Answer**: An AI Agent is an autonomous system that perceives its environment, reasons about it, and takes actions to achieve goals. The three core components are perception (sensing and interpreting input), reasoning (planning and decision-making via an LLM), and action (executing tool calls or producing output).

---

**Question**: How does the agent loop (perceive-think-act cycle) work?

**Answer**: The agent loop is a continuous cycle where the agent perceives input from the environment, sends it to an LLM for reasoning, and acts on the LLM's output (e.g., calling a tool or returning a response). The result of the action feeds back into the next perception step, creating a closed loop until the goal is achieved.

```python
while not goal_achieved:
    perception = observe_environment()
    thought = llm.reason(perception, context)
    action = parse_action(thought)
    result = execute(action)
    update_context(result)
```

---

**Question**: How do you define and call tools/functions for an LLM agent?

**Answer**: Tools are defined as JSON schemas describing the function name, parameters, and description. The LLM returns a structured tool call request, which the runtime validates and executes, then feeds the result back into the conversation.

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get the current weather for a location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {"type": "string"}
                },
                "required": ["location"]
            }
        }
    }
]
```

---

**Question**: What is function calling in LLMs and how is the tool definition format structured?

**Answer**: Function calling is an API capability where the LLM outputs a structured JSON object requesting a specific function invocation rather than generating free text. The definition includes `name`, `description`, and `parameters` following JSON Schema, allowing the model to understand when and how to call each function.

---

**Question**: How do multi-agent systems and orchestration work?

**Answer**: Multi-agent systems decompose complex tasks across multiple specialized agents that communicate and coordinate. Orchestration involves a central component that routes tasks, manages state, and handles inter-agent communication, often using patterns like supervisor, voting, or publish-subscribe.

---

**Question**: What is a Supervisor/Orchestrator agent pattern?

**Answer**: A Supervisor agent delegates subtasks to specialized worker agents, monitors their progress, and aggregates results. It decides which agent to invoke based on the task, handles errors, and can dynamically replan if a subtask fails.

```python
class SupervisorAgent:
    def run(self, task):
        plan = self.decompose(task)
        i = 0
        while i < len(plan):
            step = plan[i]
            agent = self.select_agent(step)
            result = agent.execute(step)
            if not result.success:
                plan = self.replan(step, result)
                i = 0
                continue
            i += 1
        return self.aggregate(plan)
```

---

**Question**: Explain agent delegation and handoff patterns.

**Answer**: Delegation involves one agent passing a subtask to another agent, often with context transfer. Handoff patterns include direct handoff (agent A calls agent B explicitly), router-based (a central dispatcher assigns work), and publish-subscribe (agents consume tasks from a shared queue).

---

**Question**: What is the ReAct pattern (Reasoning + Acting)?

**Answer**: ReAct interleaves reasoning traces with action steps, allowing the LLM to think before acting and observe results before the next thought. This improves transparency and correctness by making the chain-of-thought visible and actionable.

```python
def react_agent(prompt, max_iterations=10):
    for _ in range(max_iterations):
        thought = llm.generate(f"Thought: {prompt}")
        action = parse_action(thought)
        if action.type == "Finish":
            return action.final_answer
        observation = execute(action)
        prompt += f"\nObservation: {observation}"
    return "Max iterations reached"
```

---

**Question**: How does the Plan-and-Execute pattern differ from ReAct?

**Answer**: Plan-and-Execute first generates a complete step-by-step plan before taking any action, then executes each step sequentially. Unlike ReAct which interleaves thinking and acting, Plan-and-Execute commits to a plan upfront, making it better for tasks requiring predictable ordering.

```python
def plan_and_execute(task):
    plan = llm.generate(f"Create a plan for: {task}")  # returns list of steps
    i = 0
    while i < len(plan):
        step = plan[i]
        result = execute(step)
        if result.failed:
            plan = llm.generate(f"Revise plan based on: {result}")
            i = 0
            continue
        i += 1
    return result
```

---

**Question**: How do agents implement reflection and self-correction?

**Answer**: Agents reflect by feeding their own output back into the LLM with critique prompts asking for improvement or error detection. Self-correction uses this reflection to regenerate responses, retry failed actions, or adjust the plan dynamically.

```python
def reflective_agent(task):
    output = llm.generate(task)
    critique = llm.generate(f"Critique this output: {output}")
    if "error" in critique.lower():
        output = llm.generate(f"Fix the error: {critique}")
    return output
```

---

**Question**: How is memory managed in agents (short-term, long-term, episodic)?

**Answer**: Short-term memory holds the current conversation context within the LLM's window. Long-term memory persists information across sessions using vector databases or key-value stores. Episodic memory stores specific past interactions or outcomes that can be retrieved later for learning.

---

**Question**: What is the difference between vector memory and key-value memory?

**Answer**: Vector memory stores embeddings of text chunks for semantic similarity search, enabling retrieval based on meaning. Key-value memory stores structured data indexed by explicit keys, enabling exact lookups. Vector memory is used for open-ended retrieval; key-value memory is used for facts and configurations.

---

**Question**: How do you manage conversation history (windowing, summarization, hybrid)?

**Answer**: Windowing keeps only the last N messages within the context window. Summarization periodically compresses older messages into a summary. Hybrid approaches combine both: recent messages are kept verbatim while older ones are summarized, balancing context freshness with token cost.

```python
class ConversationManager:
    def __init__(self, max_tokens=4000):
        self.max_tokens = max_tokens
        self.history = []
        self.summary = ""

    def token_count(self, messages):
        return sum(len(m["content"].split()) for m in messages)  # rough estimate

    def add_message(self, msg):
        self.history.append(msg)
        if self.token_count(self.history) > self.max_tokens:
            old_messages = self.history[:len(self.history)//2]
            self.summary = llm.summarize(old_messages)
            self.history = self.history[len(self.history)//2:]

    def get_context(self):
        if self.summary:
            return [{"role": "system", "content": self.summary}] + self.history
        return self.history
```

---

**Question**: What is a Toolshed and how do you build a tool library?

**Answer**: A Toolshed is a centralized registry of available tools that agents can discover and invoke. It includes tool definitions, metadata (name, description, schema), versioning, and access control. Building one involves defining a common interface, registering tools at startup, and providing discovery mechanisms.

---

**Question**: How does tool selection and routing work in an agent system?

**Answer**: Tool selection uses the LLM's semantic understanding to match the user request to the appropriate tool based on descriptions and parameter schemas. Routing can be LLM-driven (the model chooses) or rule-based (predefined conditions map tasks to tools).

---

**Question**: How do you implement error recovery in agent workflows?

**Answer**: Error recovery uses try-catch around tool execution, with fallback logic that retries, degrades gracefully, or escalates to a human. The agent should log the error, assess whether retrying with different parameters might work, and update its context to avoid repeated failures.

```python
def safe_execute(tool, params, max_retries=3):
    for attempt in range(max_retries):
        try:
            return tool(**params)
        except TemporaryError as e:
            wait = 2 ** attempt
            time.sleep(wait)
        except FatalError as e:
            return {"error": str(e), "fallback": True}
    return escalate_to_human(params)
```

---

**Question**: Explain retry strategies with exponential backoff for agents.

**Answer**: Exponential backoff doubles the wait time between retries (1s, 2s, 4s, 8s...) to avoid overwhelming rate-limited services. Jitter adds randomness to prevent thundering herd problems. The agent should stop retrying after a max limit and escalate the failure.

```python
def retry_with_backoff(fn, max_retries=5, base_delay=1.0):
    for attempt in range(max_retries):
        try:
            return fn()
        except RateLimitError:
            delay = base_delay * (2 ** attempt) + random.uniform(0, 1)
            time.sleep(delay)
    raise MaxRetriesExceeded()
```

---

**Question**: What are guardrails for agent outputs and how do you implement them?

**Answer**: Guardrails are constraints that validate agent outputs before they reach users or systems. They include content filtering (toxicity, PII), schema validation (structured output compliance), and business rule checks. Implement them as middleware that intercepts and validates every output before delivery.

```python
class Guardrail:
    def check(self, output):
        if contains_pii(output):
            return self.redact(output)
        if not matches_schema(output):
            return self.request_regeneration(output)
        return output
```

---

**Question**: How do you implement human-in-the-loop patterns for agents?

**Answer**: Human-in-the-loop inserts a pause before high-risk actions, requesting human approval. Implementation involves returning a pending status, sending a notification to a human reviewer, and resuming once approved, rejected, or modified.

```python
class HumanInLoop:
    async def execute(self, action):
        if action.risk == "high":
            approval = await self.request_approval(action)
            if not approval.approved:
                return approval.feedback
        return await self.run(action)
```

---

**Question**: How do you implement agent observability and tracing?

**Answer**: Observability captures every step in the agent loop: LLM calls, tool invocations, decisions, errors, and latency. Tracing uses OpenTelemetry or custom spans to correlate these steps into a single trace for debugging and performance analysis.

```python
with tracer.start_as_current_span("agent_loop") as span:
    span.set_attribute("task", task_id)
    thought = llm.generate(prompt, otel_ctx=span)
    span.add_event("tool_call", {"tool": "search", "params": query})
    result = search(query)
```

---

**Question**: What is LangGraph and how does it relate to agents?

**Answer**: LangGraph is a framework for building stateful, multi-actor agent applications as directed graphs. Nodes represent LLM calls, tool executions, or decisions, and edges define control flow. It supports cycles, branching, and checkpointing natively.

```python
from langgraph.graph import StateGraph, END

graph = StateGraph(AgentState)
graph.add_node("reason", llm_node)
graph.add_node("act", tool_node)
graph.add_edge("reason", "act")
graph.add_conditional_edges("act", should_continue, {True: "reason", False: END})
```

---

**Question**: How do CrewAI and AutoGen enable multi-agent frameworks?

**Answer**: CrewAI uses role-based agents with defined goals, tasks, and tools, orchestrated by a sequential or hierarchical process. AutoGen provides conversational agents that can converse with each other and with humans, supporting group chats, nested chats, and function calls for complex workflows.

---

**Question**: How does Semantic Kernel handle agent orchestration?

**Answer**: Semantic Kernel uses a planner that auto-generates execution plans by combining plugins (functions) with AI prompts. It supports both sequential and stepwise planners, and integrates with Azure OpenAI for function calling and memory management via vector stores.

---

**Question**: What is task decomposition and how do hierarchical agents work?

**Answer**: Task decomposition breaks a complex goal into smaller, manageable subtasks that can be assigned to specialized agents. Hierarchical agents organize this into a tree where a high-level agent decomposes and delegates, and leaf agents execute atomic tasks.

---

**Question**: How do you implement checkpointing for agent state?

**Answer**: Checkpointing saves the full agent state (conversation history, variables, pending actions) to persistent storage at each step. On failure, the agent resumes from the last checkpoint rather than restarting, enabling fault tolerance for long-running workflows.

```python
class CheckpointedAgent:
    def run(self, task, checkpoint_id=None):
        state = self.load(checkpoint_id) or AgentState(task=task)
        while not state.done:
            state = self.step(state)
            self.save(state)
        return state
```

---

**Question**: How do you implement parallel agent execution?

**Answer**: Parallel execution runs multiple independent agent instances concurrently, typically using asyncio or thread pools. Use `asyncio.gather` to fan out tasks and collect results, ensuring shared state isolation to avoid race conditions.

```python
async def parallel_execute(agents, tasks):
    results = await asyncio.gather(
        *[agent.run(task) for agent, task in zip(agents, tasks)]
    )
    return results
```

---

**Question**: How do you evaluate and benchmark agent performance?

**Answer**: Evaluation uses task-specific metrics: success rate, completion time, tool call efficiency, and cost per task. Benchmarking involves running agents against curated datasets with expected outputs, using automated scoring (exact match, semantic similarity, human evaluation).

---

**Question**: What are best practices for prompt engineering in agent system prompts?

**Answer**: System prompts should define the agent's persona, available tools (with descriptions), output format constraints, and behavioral guardrails. Include few-shot examples, specify how to handle errors, and instruct the model to think step by step for complex reasoning.

```python
SYSTEM_PROMPT = """
You are a research agent. You have access to:
- search(query): Search the web
- read(url): Read a page

Always output your reasoning before calling a tool.
If a tool fails, explain the error and retry once.
Respond in this JSON format: {"thought": "...", "action": "tool_name", "params": {...}}
"""
```

---

**Question**: How do you ensure safety and alignment in agent behavior?

**Answer**: Safety involves content filtering, output validation, and restricting tool access based on permissions. Alignment ensures agent goals match user intent through prompt guardrails, constrained decoding, and human oversight for high-stakes actions.

---

**Question**: What strategies optimize costs in agent workflows?

**Answer**: Use smaller/faster models for simple tasks and larger models only for complex reasoning. Cache identical LLM calls, compress conversation history, batch independent tool calls, and set token limits per step to control spending.

---

**Question**: How do you implement caching for agent responses?

**Answer**: Cache LLM responses keyed by the concatenation of system prompt, user message, and tool results. Use a TTL-based cache (Redis or in-memory) to avoid redundant API calls for identical or semantically similar queries.

```python
import hashlib, json

cache = {}
def cached_llm_call(prompt, tools):
    key = hashlib.sha256(json.dumps({"prompt": prompt, "tools": tools}, sort_keys=True).encode()).hexdigest()
    if key in cache and not cache[key].expired:
        return cache[key].value
    result = llm.call(prompt, tools)
    cache[key] = CacheEntry(result, ttl=300)
    return result
```

---

**Question**: How do you stream agent outputs to end users?

**Answer**: Streaming sends partial tokens from the LLM response as they arrive, and streams tool call results incrementally. Use Server-Sent Events (SSE) or WebSockets to push updates, showing the agent's reasoning in real time.

```python
async def stream_agent(prompt):
    async for chunk in llm.stream(prompt):
        yield f"data: {json.dumps({'token': chunk})}\n\n"
        if chunk.tool_call:
            result = await execute_tool(chunk.tool_call)
            yield f"data: {json.dumps({'tool_result': result})}\n\n"
```

---

**Question**: How do you implement rate limiting for agent API calls?

**Answer**: Rate limiting controls the frequency of LLM and tool API calls using token-bucket or sliding-window algorithms. Apply per-user, per-agent, and global limits to prevent exceeding API quotas and manage costs.

```python
import asyncio, time

class RateLimiter:
    def __init__(self, max_rpm=60):
        self.max_rpm = max_rpm
        self.tokens = max_rpm
        self.refill_rate = max_rpm / 60.0  # tokens per second
        self.last_refill = time.monotonic()
        self._lock = asyncio.Lock()

    async def acquire(self):
        async with self._lock:
            now = time.monotonic()
            elapsed = now - self.last_refill
            self.tokens = min(self.max_rpm, self.tokens + elapsed * self.refill_rate)
            self.last_refill = now
            if self.tokens < 1:
                wait = (1 - self.tokens) / self.refill_rate
                await asyncio.sleep(wait)
                self.tokens = 0
                self.last_refill = time.monotonic()
            self.tokens -= 1
```

---

**Question**: How do you ensure structured outputs from agents?

**Answer**: Use constrained decoding or response_format parameters (e.g., `{"type": "json_object"}` in OpenAI) to force the LLM to output valid JSON or XML. Validate outputs against a Pydantic/JSON Schema model and regenerate if validation fails.

```python
response = openai.chat.completions.create(
    model="gpt-4o",
    messages=messages,
    response_format={"type": "json_object"},
    tools=tools
)
structured = json.loads(response.choices[0].message.content)
```

---

**Question**: What are agent testing strategies?

**Answer**: Test at three levels: unit tests for individual tools, integration tests for the full agent loop with mocked LLM responses, and end-to-end tests against real models using curated golden datasets. Use property-based testing for edge cases.

---

**Question**: How do you implement CI/CD for agent prompts?

**Answer**: Store prompts as version-controlled files (e.g., YAML or JSON) in the repository. Use CI pipelines to run automated evaluations against test datasets when prompts change, and promote to production only when success metrics meet thresholds.

---

**Question**: How do you version agent configurations?

**Answer**: Version the entire agent configuration (system prompt, tool definitions, model parameters, guardrails) as a single artifact. Store in source control or a registry, tag with semantic versions, and support rollback by redeploying a previous configuration.

---

**Question**: How do you monitor agent performance in production?

**Answer**: Track metrics: success rate, average steps per task, token usage, latency per step, error rate, and cost per interaction. Alert on regressions using dashboards (Grafana, Datadog) with traces correlated to each agent session.

---

**Question**: What is agent memory pruning and why is it important?

**Answer**: Memory pruning removes redundant, outdated, or low-value information from the agent's context to stay within token limits. Strategies include removing duplicate facts, summarizing old entries, and evicting least-recently-used memories.

---

**Question**: How do you build retrieval-augmented agents?

**Answer**: A retrieval-augmented agent queries a vector database for relevant documents before reasoning. It embeds the user query, retrieves top-K chunks, injects them into the LLM context, and generates a grounded response.

```python
def rag_agent(query):
    docs = vector_db.similarity_search(query, k=5)
    context = "\n".join([d.page_content for d in docs])
    prompt = f"Context:\n{context}\n\nQuestion: {query}"
    return llm.generate(prompt)
```

---

**Question**: How do you implement web browsing and API-calling agents?

**Answer**: Provide tools for HTTP requests (`GET`, `POST`), HTML parsing, and link extraction. The agent navigates by reading pages, extracting links, and following them autonomously. Implement rate limiting and respect robots.txt.

```python
tools = [
    {"name": "fetch_page", "fn": lambda url: requests.get(url).text},
    {"name": "extract_links", "fn": lambda html: BeautifulSoup(html, "html.parser").find_all("a")},
    {"name": "search", "fn": lambda q: requests.get(f"https://api.duckduckgo.com/?q={q}").json()}
]
```

---

**Question**: How do you build code execution agents safely?

**Answer**: Execute generated code in a sandboxed environment (Docker container, Pyodide, or gVisor) with restricted filesystem, network, and resource limits. Never run agent-generated code on the host machine without isolation.

```python
import subprocess
def safe_execute(code):
    result = subprocess.run(
        [
            "docker", "run", "--rm",
            "--network", "none",
            "--memory", "256m",
            "--cpus", "0.5",
            "--read-only",
            "python:slim", "python", "-c", code
        ],
        capture_output=True, text=True, timeout=30
    )
    return result.stdout
```

---

**Question**: How do you build database querying agents?

**Answer**: Provide a tool with a read-only connection that accepts SQL queries. The agent generates SQL from natural language, validates it against a schema, and returns results. Restrict to SELECT statements and apply row/query limits.

```python
import sqlparse

def query_database(sql: str) -> list | dict:
    parsed = sqlparse.parse(sql.strip())
    if len(parsed) > 1:
        return {"error": "Multiple statements are not allowed"}
    if parsed[0].get_type() != 'SELECT':
        return {"error": "Only SELECT queries are allowed"}
    with conn.cursor() as cur:
        cur.execute(sql)
        return cur.fetchall()
```

---

**Question**: How do you build email and communication agents?

**Answer**: Provide tools for listing, reading, sending, and searching emails via IMAP/SMTP APIs. The agent must handle authentication securely (OAuth2), respect rate limits, and require human confirmation before sending.

---

**Question**: How do you build research and analysis agents?

**Answer**: A research agent combines web search, document retrieval, and synthesis. It queries multiple sources, extracts key facts, cross-references, and produces a structured report with citations and confidence scores.

---

**Question**: What are swarm intelligence patterns in multi-agent systems?

**Answer**: Swarm intelligence uses many simple agents following local rules to produce emergent global behavior. Agents share information indirectly through a shared environment or blackboard, without a central controller.

---

**Question**: How do voting and consensus work in multi-agent systems?

**Answer**: Multiple agents independently evaluate the same task and vote on the best outcome. Consensus can be majority vote, weighted vote (by agent confidence), or ranked-choice. The final output is selected or aggregated based on voting results.

```python
def consensus(agents, task):
    votes = [agent.evaluate(task) for agent in agents]
    counts = {v: votes.count(v) for v in set(votes)}
    max_count = max(counts.values())
    winners = [v for v, c in counts.items() if c == max_count]
    if len(winners) > 1:
        return random.choice(winners)  # tie-breaking
    return winners[0]
```

---

**Question**: How do you implement agent specialization with roles and personas?

**Answer**: Assign each agent a distinct role (researcher, writer, reviewer) via system prompts that define its persona, expertise, and constraints. Agents collaborate by passing structured messages, each contributing their specialized capability.

---

**Question**: How do you implement fallback mechanisms in agent chains?

**Answer**: When a primary agent fails, fallback routes to a simpler or different agent with a smaller model or broader tool access. Implement cascading fallbacks (tier-1 → tier-2 → human) with degradation in capability clearly communicated.

---

**Question**: What are production deployment considerations for agents?

**Answer**: Key considerations include: idempotency of tool executions (handle duplicates safely), timeout limits per step, queue-based execution to handle load, structured logging with tracing, cost budgets with hard caps, gradual rollout with canary testing, and monitoring dashboards.

---

**Question**: How do you handle agent state persistence and recovery?

**Answer**: Persist the full agent graph state (conversation, variables, tool results) to a database or blob storage at each step. On process restart, deserialize the state and resume execution from the last saved node, ensuring at-least-once semantics.

```python
class PersistentAgent:
    def step(self, state):
        new_state = self.graph.invoke(state)
        self.db.save(state.id, new_state.model_dump())
        return new_state
```
