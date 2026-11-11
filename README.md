# New Relic plugin for GitHub Copilot

Query your New Relic telemetry from GitHub Copilot in VS Code and the Copilot CLI. Ask about errors, latency, logs, traces, alerts, deployments, and dashboards in natural language. You can also run NRQL directly.

The plugin connects Copilot to the hosted [New Relic MCP server](https://docs.newrelic.com/docs/agentic-ai/mcp/overview/) and adds skills and custom agents that guide Copilot through setup, NRQL, and incident investigation.

> The plugin is in preview.

## What you get

- **MCP server `newrelic`**: the hosted New Relic MCP server (`https://mcp.newrelic.com/mcp/`), with tools for NRQL, entities, alerts and issues, logs, traces, deployments, synthetics, and dashboards.
- **Skills**:
  - `nrsetup`: sign-in, US/EU/JP regions, access troubleshooting
  - `nrql`: write and run NRQL safely and efficiently
  - `nrinvestigate`: step-by-step incident and regression investigation
- **Custom agents** (select one from the agent picker in VS Code, or run `/agent` in the Copilot CLI):
  - `New Relic APM Agent`: errors, latency, throughput, transactions, and deployment impact
  - `New Relic Incident Response Agent`: correlates alerts, traces, errors, and deployments with your code to find root causes and suggest fixes
  - `New Relic Kubernetes Agent`: pod crashes, OOMKills, evictions, scheduling, and node pressure (needs Kubernetes data in New Relic)
  - `New Relic Financial Operations (FinOps) Agent`: cloud spend, cost anomalies, and budgets (needs Cloud Cost Intelligence)

All agents use the bundled `newrelic` MCP server.

## Requirements

- A New Relic account. MCP is available to all accounts. Some advanced tools need specific user types.
- Your New Relic administrator must have **New Relic MCP Server** and **Local Clients** enabled in feature control manager. Both are on by default. See [organization access](https://docs.newrelic.com/docs/agentic-ai/mcp/overview/#org-access).
- VS Code with GitHub Copilot, the Copilot CLI, or the Copilot app.

## Install

**Copilot CLI**

```shell
copilot plugin install new-relic@awesome-copilot
```

**VS Code**

1. Open the Extensions view.
2. Search for `@agentPlugins new-relic`.
3. Select **New Relic** and click **Install**.

**Copilot app**

1. Click **+ Add** and select **Plugin**.
2. Search for `new-relic@awesome-copilot` and install it.

## Connect

The plugin uses OAuth, so you don't need an API key.

- **VS Code**: run **MCP: List Servers**, select `newrelic`, and start it. Sign in to New Relic in the browser window that opens.
- **Copilot CLI**: run `/mcp`, select `newrelic`, and follow the sign-in prompt.
- **Copilot app**: type `/nrsetup` and follow the prompts, or ask Copilot to "set up New Relic". Sign in to New Relic in the browser window that opens.

Or run `/nrsetup` in Copilot (or just ask it to "set up New Relic") and the skill walks you through it. You can also run `/nrql` to write and run a query, and `/nrinvestigate` to investigate an incident.

**EU or JP accounts**: the bundled server uses the US endpoint. Ask Copilot to "set up New Relic for the EU region", or see the `nrsetup` skill. EU uses `https://mcp.eu.newrelic.com/mcp/` and JP uses `https://mcp.jp.newrelic.com/mcp/`.

## Try it

- "What New Relic issues are open right now?"
- "Show error rate and p95 latency for checkout-api over the last hour, compared with yesterday"
- "Did today's deployment of payments-service cause the latency spike?"
- "Run NRQL: `SELECT count(*) FROM Transaction FACET appName SINCE 1 hour ago`"
- "Find recent error logs for the cart service and summarize the top causes"

## Security

- The MCP server acts as the signed-in user and never grants more than that user's roles allow. Use an account with least-privilege access.
- Your MCP client stores OAuth tokens. Treat them like passwords. Access tokens expire after about 5 minutes and refresh automatically. You sign in again after about 24 hours.
- Administrators can revoke access for one user, or turn off MCP for the whole organization in feature control manager.
- NRQL sent through MCP is recorded in `NrdbQuery` with `productComponent = 'New Relic AI MCP Server'` for auditing.

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `newrelic` tools are missing | Sign in. See [Connect](#connect). |
| 401 errors after it worked before | The session has expired. Sign in again. |
| 403 or "access disabled" | Ask your administrator to check feature control manager and your role. |
| Tools load but return no data | Check the account region (EU/JP) and the account ID. |

More help is in the [New Relic MCP troubleshooting docs](https://docs.newrelic.com/docs/agentic-ai/mcp/troubleshoot/).

## Plugin layout

This plugin follows the [Agent Plugins 1.0.0](https://agent-plugins.org/) format:

```
plugin.json                 # manifest
mcp.json                    # hosted New Relic MCP server (OAuth, no secrets)
com.github.copilot/
  agents/                   # custom agents (Copilot reads them from this fixed path)
    new-relic-apm.agent.md
    new-relic-incident-response.agent.md
    new-relic-kubernetes.agent.md
    new-relic-finops.agent.md
skills/
  nrsetup/
  nrql/
  nrinvestigate/
```

## Support

- Docs: https://docs.newrelic.com/docs/agentic-ai/mcp/overview/
- Issues: https://github.com/newrelic-partners/Github-Copilot-plugin/issues

## License

Apache-2.0
