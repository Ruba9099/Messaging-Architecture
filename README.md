# Redis Pub/Sub - Python Backend

A simple, modular Python backend demonstrating Redis Publish/Subscribe.

## Project Structure

```
redis_pubsub_project/
├── config.py               # Redis connection settings (host, port, db, channel)
├── messaging/
│   ├── __init__.py
│   ├── interface.py        # Abstract MessageBroker interface (contract)
│   └── redis_broker.py     # RedisBroker: Redis implementation of the interface
├── publisher.py             # Sends messages
├── subscriber.py            # Receives and prints messages
├── requirements.txt
└── README.md
```

## Why the `messaging` abstraction?

`publisher.py` and `subscriber.py` never import `redis` directly. They only
depend on `MessageBroker`, an abstract interface with two methods:
`publish()` and `subscribe()`.

`RedisBroker` is the only class that actually knows about Redis.

If the project later needs to switch to Kafka or RabbitMQ, you only write a
new class (e.g. `KafkaBroker`) that implements the same `MessageBroker`
interface — `publisher.py` and `subscriber.py` stay untouched.

## 1. Install prerequisites

Install Python 3 and Docker Desktop, then start Docker Desktop. In VS Code,
open this folder and run **Terminal: Run Build Task** (or press `Ctrl+Shift+B`)
to create the `.venv` environment and install the Python dependency. The
recommended Python, Pylance, and Container Tools extensions are listed in
`.vscode/extensions.json`.

## 2. Run Redis with Docker

```bash
docker run --rm -p 6379:6379 --name redis-pubsub redis:7-alpine
```

Alternatively, run the **Start Redis (Docker)** task in VS Code.

## 3. Run the subscriber and publisher

```bash
.\.venv\Scripts\python.exe subscriber.py
```

In another terminal:

```bash
.\.venv\Scripts\python.exe publisher.py
```

Type any message in the publisher terminal and press Enter. It will
instantly appear in the subscriber terminal's console output.

Type `exit` in the publisher to stop it. Use `Ctrl+C` to stop the subscriber.
You can also select **Publisher + Subscriber** from the VS Code Run and Debug
view to launch both processes.

## Expected Flow

```
Python Publisher  --publish-->  Redis  --broadcast-->  Python Subscriber  --> Console Output
```

## Testing with multiple messages

With the subscriber running, send several messages one after another from
the publisher terminal (e.g. `hello`, `test 1`, `test 2`). Each one should
appear immediately in the subscriber's console, confirming Pub/Sub is
working correctly.

---

## Part 2: Two AI agents talking to each other

Built on top of the same `messaging` abstraction. Two agents, each in its
own terminal, exchange messages over Redis automatically — no human typing
needed after the first message.

```
agents/
├── base_agent.py   # AIAgent class: listen -> think -> reply
└── brain.py        # the "thinking" logic (rule-based, swappable)
agent_a.py            # speaks first
agent_b.py            # waits, then replies
```

### How it works

- `AgentA` listens on channel `agent_a_channel` and replies on `agent_b_channel`.
- `AgentB` listens on channel `agent_b_channel` and replies on `agent_a_channel`.
- Each message is a small JSON object: `{"sender": ..., "text": ..., "turn": ...}`.
- When a message arrives, the agent waits 1 second (simulated "thinking"),
  picks a reply using `brain.py`, and publishes it back.
- The conversation automatically stops after `max_turns` (default 6) so it
  doesn't run forever.

### Run it

Terminal 1:
```bash
python agent_b.py
```

Terminal 2:
```bash
python agent_a.py
```

Press Enter in AgentA's terminal to start. Watch both terminals — you'll
see the two agents exchange messages back and forth automatically until
`max_turns` is reached.

### Swapping in a real LLM later

`brain.py` currently uses simple keyword matching so the demo needs zero
external setup. To use a real AI model instead, write a new function with
the same signature — `def smart_brain(incoming_text: str) -> str` — that
calls the Anthropic API (or any LLM API) and returns its reply, then pass
`brain=smart_brain` instead of `brain=simple_brain` when creating the
`AIAgent`. Nothing else in the project needs to change.
