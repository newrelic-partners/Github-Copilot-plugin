---
name: nrsetup
description: Connects and troubleshoots the New Relic MCP server `newrelic` (OAuth sign-in, EU/JP regions, access errors). Use when the user wants to set up or connect New Relic, when a New Relic request is made but `newrelic` tools are missing from your tool list, or when New Relic tools fail with 401, 403, or connection errors.
license: Apache-2.0
---

# New Relic MCP setup

The New Relic MCP server bundled with this plugin is named `newrelic`. It points at the US endpoint `https://mcp.newrelic.com/mcp/` and authenticates with OAuth, so no API key is needed.

If `newrelic` tools are not in your tool list, do not conclude New Relic is unavailable. It usually means the user has not signed in yet, or their account is in a different region. Work through the steps below.

Never ask the user to paste an API key, token, or password into the chat.

## Step 1: Check the current state

- **Tools are present** (for example `execute_nrql_query`, `list_available_new_relic_accounts`): call `list_available_new_relic_accounts`. If it returns accounts, setup is complete. Continue with the user's original request without mentioning this check.
- **Tools are missing, or calls fail with 401/unauthorized**: go to Step 2.
- **Calls fail with 403/forbidden or "MCP access disabled"**: go to Step 4.

## Step 2: Sign in with OAuth

Tell the user to sign in to New Relic for the `newrelic` MCP server:

- **VS Code**: open the Command Palette, run **MCP: List Servers**, select `newrelic`, then choose **Start Server** (or **Restart Server**). A browser window opens for the New Relic login. After signing in, return to VS Code.
- **Copilot CLI**: run `/mcp`, select `newrelic`, and follow the sign-in prompt. If no prompt appears, restart the Copilot CLI session.

Access tokens refresh automatically. Users must sign in again after about 24 hours, or if an administrator revokes access.

After the user confirms they signed in, repeat Step 1.

## Step 3: Accounts outside the US region

The bundled server uses the US endpoint. Ask the user which region their New Relic account is in if sign-in succeeds but no accounts or data appear, or if they mention EU or Japan.

| Region | MCP endpoint |
|--------|--------------|
| US (default) | `https://mcp.newrelic.com/mcp/` |
| EU | `https://mcp.eu.newrelic.com/mcp/` |
| JP | `https://mcp.jp.newrelic.com/mcp/` |

For EU or JP, the user adds their own server entry with the regional URL and disables the bundled `newrelic` server so the two do not conflict:

- **VS Code**: run **MCP: Open User Configuration** and add:

  ```json
  {
    "servers": {
      "newrelic-eu": {
        "type": "http",
        "url": "https://mcp.eu.newrelic.com/mcp/"
      }
    }
  }
  ```

  Use the JP URL instead if applicable. Then run **MCP: List Servers**, start `newrelic-eu`, and sign in. Stop the bundled `newrelic` server.

- **Copilot CLI**: run `/mcp add`, choose HTTP, name it `newrelic-eu`, and enter the regional URL. Then sign in through `/mcp`.

From then on, use the regional server's tools instead of the bundled `newrelic` server.

## Step 4: Access is blocked by the organization

A 403, or sign-in that succeeds without tools, usually means MCP access is not enabled for the user. Explain that a New Relic administrator must check:

1. **Feature control manager**: the **New Relic MCP Server** and **Local Clients** toggles must be on. They are on by default. **Preview Tools** is needed for preview tools such as `search_traces`.
2. **Roles**: the user's group needs an organization role that can read the MCP server, such as Organization Read Only, Organization Manager, or Organization Product Admin.

Point the user to https://docs.newrelic.com/docs/agentic-ai/mcp/overview/#org-access for details.

## API key fallback (only if OAuth is not an option)

OAuth is recommended. Some tools only work with OAuth, including `natural_language_to_nrql_query`, `analyze_deployment_impact`, `generate_alert_insights_report`, and `generate_user_impact_report`. API keys also require an administrator to enable the **API Keys** toggle in feature control manager.

If the user still wants an API key, they need a **User** key (it starts with `NRAK-`). Have them configure it in their client's own settings so the key never appears in chat:

- **VS Code**: in the user MCP configuration, add an `api-key` header that reads from a secret input:

  ```json
  {
    "inputs": [
      {
        "type": "promptString",
        "id": "newrelic-api-key",
        "description": "New Relic User API key (NRAK-...)",
        "password": true
      }
    ],
    "servers": {
      "newrelic-apikey": {
        "type": "http",
        "url": "https://mcp.newrelic.com/mcp/",
        "headers": { "api-key": "${input:newrelic-api-key}" }
      }
    }
  }
  ```

- **Copilot in the cloud (github.com)**: store the key as a personal Copilot agent secret whose name starts with `COPILOT_MCP_`. Then add a custom MCP server under **Customize** with the header `api-key: $COPILOT_MCP_<NAME>`. Leave the timeout field empty.

## Optional: limit which tools load

The server accepts an `include-tags` HTTP header (for example `discovery,alerting`) to load only some tool groups. Users can set it on their own server entry. See https://docs.newrelic.com/docs/agentic-ai/mcp/tool-reference/ for the available tags.

## Troubleshooting

- **Worked yesterday, fails today**: the OAuth session has expired. Repeat Step 2.
- **Tools load but queries return no data**: wrong region (Step 3), or the wrong account. Call `list_available_new_relic_accounts` and confirm the account ID with the user.
- **More help**: https://docs.newrelic.com/docs/agentic-ai/mcp/troubleshoot/
