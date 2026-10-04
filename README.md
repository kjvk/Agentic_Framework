# Autonomous IT Support Agent

A Python notebook that demonstrates a Level 1 IT incident responder using OpenAI function calling. The agent investigates reported server issues, calls simulated support tools, and summarizes its findings.

## How it works

1. A user describes an incident and identifies a server.
2. The model selects tools to inspect server health and recent logs.
3. Python executes the selected functions and returns their results to the model.
4. The model can request more tools or provide a final response.

The notebook's system prompt directs the agent to restart services when CPU or memory usage exceeds 90%, and to escalate critical dependency errors that a restart would not resolve.

## Tools

| Tool | Purpose |
| --- | --- |
| `get_server_health(server_id)` | Returns simulated CPU usage, memory usage, and server status. |
| `fetch_recent_logs(server_id, lines=5)` | Returns log entries from a predefined dataset. |
| `restart_service(server_id)` | Simulates a service restart and returns a JSON confirmation. |
| `escalate_to_engineer(summary)` | Simulates creating an escalation ticket and returns a JSON confirmation. |

All tools return JSON strings. The execution loop associates each result with its original `tool_call_id` so the model can interpret the outcome.

## Requirements

- Google Colab (the notebook uses `google.colab.userdata`)
- The `openai` Python package
- An OpenAI API key with access to `gpt-4.1-mini`

## Getting started

1. Open `Assignment_completed.ipynb` in Google Colab.
2. Install the dependency in a notebook cell if needed:

   ```python
   %pip install openai
   ```

3. In Colab's **Secrets** panel, add a secret named `OPENAI_APIKEY` and enable notebook access.
4. Run the cells in order to initialize the client, define the tools and schemas, and define `run_it_agent`.
5. Run an individual scenario cell, or run all scenario cells to try the supplied examples.

The notebook initializes its client with:

```python
from openai import OpenAI
from google.colab import userdata

client = OpenAI(api_key=userdata.get("OPENAI_APIKEY"))
```

Do not commit API keys to the repository. Running the agent makes API requests and may incur usage charges.

### Running outside Colab

To use a local Jupyter environment, remove the `google.colab` import and replace client initialization with:

```python
import os
from openai import OpenAI

client = OpenAI(api_key=os.environ["OPENAI_API_KEY"])
```

Install `openai` in the notebook's Python environment and set the `OPENAI_API_KEY` environment variable before starting Jupyter.

## Example

After running the setup and function-definition cells:

```python
run_it_agent("The payment-server-01 is extremely slow and timing out.")
```

The notebook prints tool activity and the agent's final response. The exact tool sequence and wording are selected by the model and may vary.

## Included scenarios

| Server | Simulated condition | Intended behavior |
| --- | --- | --- |
| `payment-server-01` | CPU at 98%; thread exhaustion and timeouts | Restart the service. |
| `db-node-02` | Healthy metrics and normal logs | Report findings or request more incident details. |
| `auth-service-03` | Memory at 95%; out-of-memory errors | Restart the service according to the notebook's policy. |
| `search-index-09` | Connection refused; search dependency unavailable | Escalate to an engineer. |
| `frontend-node-04` | Healthy metrics and successful requests | Report healthy status; no corrective action needed. |

## Completed assignment sections

- Implemented `restart_service`.
- Implemented `escalate_to_engineer`.
- Added the restart tool description and `server_id` parameter schema.
- Added the escalation tool's `summary` parameter schema.
- Added tool-result messages to the agent's conversation history.

## Validation

The completed notebook passed Python syntax checks and a simulated tool-call round trip covering restart and escalation results. Live OpenAI API execution was not part of that validation.

## Limitations

This is an educational simulation. It does not connect to real servers, restart real services, or create real support tickets. Health metrics and logs are static and do not change after a simulated restart.

The decision policy is expressed in the system prompt rather than enforced by Python checks. The execution loop has no iteration limit, and API failures or malformed tool arguments are not handled. These areas would need additional safeguards before adapting the example for operational use.

## Reference

[OpenAI function calling documentation](https://developers.openai.com/api/docs/guides/function-calling)
