# Publishing the New Relic Copilot plugin

This is an internal checklist for getting the plugin listed in the `awesome-copilot` marketplace, the same way Datadog and Sumo Logic are listed. You can delete this file before the repository goes public.

## 1. Before publishing

- [ ] **Open-source approval**: confirm the repository can be public under `newrelic-partners`, with the Apache-2.0 license and a `LICENSE` file (check with opensource@newrelic.com).
- [x] **Add the `LICENSE` file**: the standard Apache-2.0 text.
- [ ] **Add a logo (optional)**: e.g. `logo.png`, referenced from the README.
- [ ] **Test locally in VS Code**: install from the local folder, sign in with OAuth, and run a few prompts from the README.
- [ ] **Test in the Copilot CLI** (no install needed; loads the folder for one session):
  ```shell
  cd newrelic-copilot-plugin
  copilot --plugin-dir .         # then /mcp -> newrelic -> sign in
  copilot --plugin-dir . plugin list   # should show new-relic under External Plugins
  ```
  Then check that the four agents appear in `/agent` and that each can call `newrelic` tools.
- [ ] **Test an EU account** (if available) with the `nrsetup` skill's region steps.

## 2. Create the repository and tag a release

```shell
cd newrelic-copilot-plugin
git init && git add . && git commit -m "New Relic plugin for GitHub Copilot v0.1.0"
git branch -M main
git remote add origin https://github.com/newrelic-partners/copilot-plugin.git
git push -u origin main
git tag v0.1.0 && git push origin v0.1.0
git rev-parse v0.1.0^{commit}      # full 40-character SHA for the submission
```

The `version` in `plugin.json` must match the version submitted.

## 3. Submit to awesome-copilot

Do **not** open a pull request that edits `plugins/external.json`. Open a new issue in https://github.com/github/awesome-copilot/issues using the **external plugin** issue form, and fill in:

| Field | Value |
|-------|-------|
| Plugin name | `new-relic` |
| Short description | Use New Relic directly in GitHub Copilot through the hosted New Relic MCP server. Query telemetry with NRQL, investigate errors, alerts, and deployments, and explore logs, traces, and dashboards in natural language. |
| GitHub repository | `newrelic-partners/copilot-plugin` |
| Plugin path | `/` (repository root) |
| Ref | `v0.1.0` |
| Commit SHA | *(output of `git rev-parse v0.1.0^{commit}`)* |
| Plugin version | `0.1.0` |
| License | `Apache-2.0` |
| Author name | New Relic |
| Author URL | https://newrelic.com |
| Homepage URL | https://docs.newrelic.com/docs/agentic-ai/mcp/overview/ |
| Keywords | new-relic, observability, monitoring, apm, nrql, logs, traces, alerts, incidents, mcp |
| Notes for reviewers | Official New Relic plugin. It bundles the hosted New Relic MCP server (OAuth via MCP authorization discovery, no secrets in config) and three Agent Skills for setup, NRQL, and incident investigation, and four custom agents (APM, incident response, Kubernetes, FinOps). |

## 4. What happens next

Automation runs these gates. All of them must pass:

- Agent Plugins 1.0.0 manifest compliance (strict `plugin.json`)
- `vally lint`, including Agent Skills spec compliance and file references
- An install smoke test through the Copilot CLI
- A version and ref/SHA consistency check

If a gate fails, the issue is labeled `requires-submitter-fixes`. Fix the problem, push a new tag, update the issue, and comment `/rerun-intake`. Once a maintainer comments `/approve`, a bot opens the pull request that adds the listing. Listings are re-reviewed every 6 months.

## 5. Shipping updates

Bump `version` in `plugin.json`, then tag, push, and open a pull request against `github/awesome-copilot` that updates our entry in `plugins/external.json` with the new `ref`, `sha`, and `version`. The same quality gates run on that pull request.

## Reference implementations

- Datadog: https://github.com/datadog-labs/copilot-plugin (uses an older manifest style)
- Sumo Logic: https://github.com/SumoLogic/sumologic-ai-plugins (Agent Plugins 1.0.0, closest to ours)
