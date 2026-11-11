---
name: nrinvestigate
description: Investigates production incidents with New Relic data, covering error spikes, latency regressions, open alerts and issues, bad deployments, and failing dependencies, using the `newrelic` MCP tools. Use when the user asks why a service is slow or erroring, what is alerting, whether a deploy caused a problem, or for a root-cause summary of an incident.
license: Apache-2.0
---

# Incident investigation with New Relic

Use the `newrelic` MCP tools. If they are missing from your tool list, follow the `nrsetup` skill first.

Work from broad to specific, and stop when you have enough evidence to answer. Tell the user what you checked as you go, in one line per step, not a wall of output.

## 1. Establish scope

- Confirm the account with `list_available_new_relic_accounts` if it isn't known.
- Identify the affected entity with `search_entity_with_tag` or `get_entity`. Note its GUID; most tools work best with it.
- Agree on the time window. Default to the last hour, or the window around the alert start time.

## 2. What is happening right now?

- **Open issues and incidents**: `list_recent_issues`, `search_incident`
- **Health snapshot**: `analyze_golden_metrics` for the entity (latency, throughput, error rate, saturation)
- **Alert context**: `list_alert_policies` and `list_alert_conditions`. If OAuth is available, `generate_alert_insights_report` gives a summary.

## 3. Pick the matching path

**Errors are up**
1. `analyze_errors` and `list_entity_error_groups` show the top error classes and messages.
2. `search_errors` finds specific occurrences.
3. `analyze_entity_logs` or `list_recent_logs` shows matching log lines.
4. `search_traces` and `get_distributed_trace_details` find an example failing trace (preview tools).

**Latency is up**
1. `analyze_transactions` finds the slowest transactions.
2. Run `execute_nrql_query` with `COMPARE WITH 1 day ago` to quantify the regression.
3. `analyze_trace_anomaly` and `analyze_service_dependencies` check downstream services and datastores (preview tools).
4. `analyze_threads` and `list_garbage_collection_metrics` cover JVM and runtime-level suspects.

**It started after a change**
1. `list_change_events` shows deployments and other changes in the window.
2. `analyze_deployment_impact` gives a before/after comparison (requires OAuth).

**A dependency looks unhealthy**
1. `list_related_entities` and `analyze_service_dependencies` map the dependency chain.
2. Repeat step 2 for the most suspicious upstream or downstream entity.

**Synthetic checks are failing**
1. `list_synthetic_monitors`, `list_synthetic_monitor_results`, and `get_synthetic_check_statistics`.

When a specialized tool is not available (for example, preview tools are disabled or the user signed in with an API key), fall back to NRQL with `execute_nrql_query`. Proven query sequences are in [references/troubleshooting-workflows.md](references/troubleshooting-workflows.md).

## 4. Report

End with a short summary:

- **What's wrong**: the symptom, with numbers (for example, "error rate rose from 0.4% to 6.1% at 14:05 UTC").
- **Most likely cause**: the evidence behind it (error class, deploy revision, slow dependency).
- **Confidence**: high, medium, or low, and what would confirm it.
- **Next steps**: concrete actions, such as rolling back a revision or checking a specific dependency.

If the user is working in a repository, offer to connect the findings to the code: the files behind the failing transaction, or the commit in the suspect deployment. `analyze_github_newrelic_gaps` can help spot services in the repo that are missing instrumentation.

Do not guess at causes the data does not support. Say what is unknown.
