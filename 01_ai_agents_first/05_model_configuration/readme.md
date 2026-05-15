# [How to configure LLM Providers at different levels (Global, Run and Agent)](https://colab.research.google.com/drive/1nWQny-AxpqQB3HGkyFLBCuai3G4igx6X?usp=sharing)?
---

## Astraflow Provider Configuration

[Astraflow](https://astraflow.ucloud-global.com) by UCloud is a fully OpenAI-compatible platform supporting **200+ models** through a single API key. Because it is OpenAI-compatible, you configure it the same way as any other third-party provider shown below — just set `base_url` and `api_key`.

**Get your API key**: https://astraflow.ucloud-global.com (global) | https://astraflow.ucloud.cn (China)

### Astraflow — Agent Level

```python
import os
import asyncio
from openai import AsyncOpenAI
from agents import Agent, OpenAIChatCompletionsModel, Runner, set_tracing_disabled

# Global endpoint — use https://api.modelverse.cn/v1 + ASTRAFLOW_CN_API_KEY for China
astraflow_client = AsyncOpenAI(
    api_key=os.environ["ASTRAFLOW_API_KEY"],
    base_url="https://api-us-ca.umodelverse.ai/v1",
)

set_tracing_disabled(disabled=True)

async def main():
    agent = Agent(
        name="AstraflowAssistant",
        instructions="You only respond in haikus.",
        model=OpenAIChatCompletionsModel(
            model="gpt-4o",   # Any of 200+ models supported by Astraflow
            openai_client=astraflow_client,
        ),
    )
    result = await Runner.run(agent, "Tell me about recursion in programming.")
    print(result.final_output)

if __name__ == "__main__":
    asyncio.run(main())
```

### Astraflow — Run Level

```python
import os
from agents import Agent, Runner, AsyncOpenAI, OpenAIChatCompletionsModel
from agents.run import RunConfig

astraflow_client = AsyncOpenAI(
    api_key=os.environ["ASTRAFLOW_API_KEY"],
    base_url="https://api-us-ca.umodelverse.ai/v1",
)

model = OpenAIChatCompletionsModel(
    model="gpt-4o",
    openai_client=astraflow_client,
)

config = RunConfig(
    model=model,
    model_provider=astraflow_client,
    tracing_disabled=True,
)

agent = Agent(name="AstraflowAssistant", instructions="You are a helpful assistant")
result = Runner.run_sync(agent, "Hello, how are you?", run_config=config)
print(result.final_output)
```

### Astraflow — Global Level

```python
import os
from agents import (
    Agent, Runner, AsyncOpenAI,
    set_default_openai_client, set_tracing_disabled, set_default_openai_api,
)

set_tracing_disabled(True)
set_default_openai_api("chat_completions")

astraflow_client = AsyncOpenAI(
    api_key=os.environ["ASTRAFLOW_API_KEY"],
    base_url="https://api-us-ca.umodelverse.ai/v1",
)
set_default_openai_client(astraflow_client)

agent = Agent(
    name="AstraflowAssistant",
    instructions="You are a helpful assistant",
    model="gpt-4o",   # Any of 200+ models supported by Astraflow
)

result = Runner.run_sync(agent, "Hello")
print(result.final_output)
```

> **China users**: replace `base_url` with `https://api.modelverse.cn/v1` and use `os.environ["ASTRAFLOW_CN_API_KEY"]`.

---


Agents SDK is setup to use OpenAI as default providers. When using other providers you can setup at different levels:
1. Agent Level
2. RUN LEVEL
3. Global Level

We will always your Agent Level Configuration so each agent can use the LLM best fit for it.

### 1. AGENT LEVEL

```python
import asyncio
from openai import AsyncOpenAI
from agents import Agent, OpenAIChatCompletionsModel, Runner, set_tracing_disabled

gemini_api_key = ""

#Reference: https://ai.google.dev/gemini-api/docs/openai
client = AsyncOpenAI(
    api_key=gemini_api_key,
    base_url="https://generativelanguage.googleapis.com/v1beta/openai/",
)

set_tracing_disabled(disabled=True)

async def main():
    # This agent will use the custom LLM provider
    agent = Agent(
        name="Assistant",
        instructions="You only respond in haikus.",
        model=OpenAIChatCompletionsModel(model="gemini-2.0-flash", openai_client=client),
    )

    result = await Runner.run(
        agent,
        "Tell me about recursion in programming.",
    )
    print(result.final_output)


if __name__ == "__main__":
    asyncio.run(main())
```

### 2. RUN LEVEL

```python
from agents import Agent, Runner, AsyncOpenAI, OpenAIChatCompletionsModel
from agents.run import RunConfig

gemini_api_key = ""

#Reference: https://ai.google.dev/gemini-api/docs/openai
external_client = AsyncOpenAI(
    api_key=gemini_api_key,
    base_url="https://generativelanguage.googleapis.com/v1beta/openai/",
)

model = OpenAIChatCompletionsModel(
    model="gemini-2.0-flash",
    openai_client=external_client
)

config = RunConfig(
    model=model,
    model_provider=external_client,
    tracing_disabled=True
)

agent: Agent = Agent(name="Assistant", instructions="You are a helpful assistant")

result = Runner.run_sync(agent, "Hello, how are you.", run_config=config)

print(result.final_output)
```

### GLOBAL

```python
from agents import Agent, Runner, AsyncOpenAI, set_default_openai_client, set_tracing_disabled, set_default_openai_api

gemini_api_key = ""
set_tracing_disabled(True)
set_default_openai_api("chat_completions")

external_client = AsyncOpenAI(
    api_key=gemini_api_key,
    base_url="https://generativelanguage.googleapis.com/v1beta/openai/",
)
set_default_openai_client(external_client)

agent: Agent = Agent(name="Assistant", instructions="You are a helpful assistant", model="gemini-2.0-flash")

result = Runner.run_sync(agent, "Hello")

print(result.final_output)
```
