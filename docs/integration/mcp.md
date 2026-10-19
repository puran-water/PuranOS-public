# Connect a customer’s LLM

The public Puran Water MCP endpoint is:

```text
https://intake.puranwater.com/mcp
```

Transport: **Streamable HTTP**. Authentication: **none for public discovery tools**. Add it to a compatible MCP client; the website provides installation controls and client configuration examples at [Connect your LLM](https://revamp.puranwater-com.pages.dev/connect/).

For Claude Code:

```sh
claude mcp add --transport http puran-water https://intake.puranwater.com/mcp
```

## Work directly in the connected conversation

Ask the client to use `describe_engagements`, identify your project stage, then call `check_fit` and `data_checklist`. The results return to your agent. No submission or booking is needed to use these tools.

| Tool | Behavior |
| --- | --- |
| `describe_engagements` | Returns published service stages and engagement boundaries. |
| `check_fit` | Applies deterministic scope rules to the supplied stage/category. |
| `data_checklist` | Returns the preparation list for a project stage. |
| `book_call` | Returns the existing booking URL; it does not book an appointment. |
| `submit_brief` | Optional consent-based submission. Requires email confirmation and browser review before delivery; availability depends on server status. |

Resources are `puranwater://engagements` and `puranwater://intake-privacy`. The server also exposes the `prepare_project_brief` prompt. Clients should discover the actual schemas and honor returned errors and availability.

The optional submission action is separate from read-only discovery. It must not be initiated without the user’s instruction.

## Scope

The public server does not provide engineering simulations, project documents, private databases or plant-control access. It is not a proxy to the internal MCP fleet. Customer engineering access is separately scoped and authorized for the engagement.

[Server metadata](https://revamp.puranwater-com.pages.dev/connect/server.json) is publisher-provided discovery information. Publication in the official MCP Registry remains pending.

## Verification

Read-only live checks on October 19, 2026 verified tool discovery, resources, prompt retrieval and read-tool calls using current and legacy MCP clients. No brief was submitted and no email was sent by these checks. This verifies the public discovery connection, not the delivery of an optional submission.

[Deliverable integration](deliverables.md) · [OpenAI client setup](https://developers.openai.com/api/docs/guides/custom-mcp-server) · [Claude Code setup](https://code.claude.com/docs/en/mcp)
