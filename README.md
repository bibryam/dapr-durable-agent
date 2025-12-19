# Dapr Agents Hello World

Minimal setup to run a Dura
bleAgent with Dapr Agent.

## Prerequisites
- [An OpenAI API key](https://platform.openai.com/api-keys) or another supported LLM
- [Python 3.10+](https://www.python.org/downloads/)
- [Docker](https://docs.docker.com/desktop/)
- [Dapr CLI](https://docs.dapr.io/getting-started/install-dapr-cli/)

## Setup
- Create a virtual environment:
```bash
python3.10 -m venv .venv && source .venv/bin/activate
````

* Install dependencies:

```bash
pip install -r requirements.txt
```

* Add your OpenAI or other LLM provider key to `resources/ll-provider.yaml`.

## Run the agent

```bash
dapr run --app-id agent --resources-path ./resources -p 8001 --log-level warn -- python agent_service.py
```

## What DurableAgent is doing

* Exposes the agent on `http://localhost:8001/run`
* Uses the [Dapr Conversation API](https://docs.dapr.io/developing-applications/building-blocks/conversation/) to talk to an OpenAI model
* Uses the [Dapr Pub/Sub API](https://docs.dapr.io/developing-applications/building-blocks/pubsub/) to subscribe to Redis on `assistant.topic`
* Uses the [Dapr Workflow API](https://docs.dapr.io/developing-applications/building-blocks/workflow/) as a durable execution engine to persist agent actions into Redis
* Uses the [Dapr State Store API](https://docs.dapr.io/developing-applications/building-blocks/state-management/) to persist conversation history to Redis
* Uses the Dapr State Store API to register itself in Redis for discovery by other agents
* Uses [Dapr observability features](https://docs.dapr.io/operations/observability/tracing/tracing-overview/) to send execution traces to Zipkin at `http://localhost:9411/`

## Trigger the agent over HTTP

In a separate terminal, prompt the agent over HTTP:

```bash
curl -i -X POST http://localhost:8001/run \
  -H "Content-Type: application/json" \
  -d '{"task": "Write a haiku about programming."}'
```

## Trigger the agent over PubSub

Prompt the agent by publishing the prompt to a PubSub topic:

```bash
dapr publish --publish-app-id agent --pubsub agent-pubsub --topic assistant.topic --data '{"task": "Write a haiku about programming."}'
```

## Examine workflow executions

In a separate terminal, launch the [Diagrid Dashboard](https://www.diagrid.io/blog/improving-the-local-dapr-workflow-experience-diagrid-dashboard):

```bash
docker run -p 8080:8080 ghcr.io/diagridio/diagrid-dashboard:latest
```
View local workflow runs at `http://localhost:8080/`

![diagrid-dashboard.png](images/diagrid-dashboard.png)

## Examine agent traces in Zipkin

The Dapr CLI installs and runs Zipkin by default. View agent traces at: `http://localhost:9411/`

If Zipkin is not running, start it with:

```bash
docker run -d -p 9411:9411 openzipkin/zipkin
```
![zipkin.png](images/zipkin.png)

## Examine Redis storage

Launch Redis Insight:

```bash 
docker run --rm -d --name redisinsight -p 5540:5540 redis/redisinsight:latest
```

View Redis Insight at: `http://localhost:5540/`. Connect to local Redis at `host.docker.internal:6379` (Mac or Windows) or `172.17.0.1:6379` (Linux). Inspect persisted conversation history, workflow execution, and PubSub messages.

![redis-insights.png](images/redis-insights.png)

## Next Steps

For a detailed visualization of agent workflow execution, follow [this quickstart guide](https://diagrid.ws/durable-agent-qs)

![diagrid-catalyst.png](images/diagrid-catalyst.png)
