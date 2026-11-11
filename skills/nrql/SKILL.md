---
name: nrql
description: Writes, runs, and explains NRQL (New Relic Query Language) queries against New Relic telemetry using the `newrelic` MCP tools. Use when the user asks for metrics, counts, latency, error rates, logs, spans, or any New Relic data, mentions NRQL, or asks to convert a question into a New Relic query.
license: Apache-2.0
---

# NRQL with the New Relic MCP server

Use the `newrelic` MCP tools to answer data questions. If they are missing from your tool list, follow the `nrsetup` skill first.

## Workflow

1. **Pick the account.** If the user has not given an account ID, call `list_available_new_relic_accounts`. If there is more than one account, ask which to use.
2. **Find the entity name.** If the user names a service loosely ("checkout"), confirm the exact `appName` or `service.name` with `search_entity_with_tag` or `get_entity` before querying.
3. **Write the query.** Follow the rules below. For complex natural-language questions you can also call `natural_language_to_nrql_query`. Review its output before running it. This tool requires OAuth.
4. **Convert times when needed.** If a tool wants epoch milliseconds, use `convert_time_period_to_epoch_ms` instead of calculating by hand.
5. **Run it** with `execute_nrql_query`.
6. **Report.** Show the NRQL you ran in a code block, then summarize the result in plain language (numbers, trends, top offenders). Suggest one useful follow-up query.

## Query rules

- **Always set a time range.** Start narrow (`SINCE 30 minutes ago` or `SINCE 1 hour ago`) and widen only when needed.
- **Always scope.** Filter by `appName`, `service.name`, or `entity.guid` when the user's question is about one service.
- **Always `LIMIT`** raw-event queries (`LIMIT 100` is a good default). Avoid `SELECT *` unless the user needs raw events.
- **Aggregate over long ranges.** Use `count`, `average`, `percentile`, and `TIMESERIES` instead of raw rows.
- **Only `FACET` on low-cardinality attributes** (`name`, `appName`, `host`), never on `traceId` or user IDs.
- **Backtick attribute names with dots:** `` `service.name` ``, `` `http.status_code` ``.
- **Use `COMPARE WITH`** for "is it worse than yesterday?" questions instead of running two queries.

## Common starting points

```sql
-- Error rate and throughput for one service
SELECT percentage(count(*), WHERE error IS true) AS 'Error rate %', count(*) AS 'Requests'
FROM Transaction WHERE appName = 'checkout-api' SINCE 1 hour ago

-- p50/p95/p99 latency compared with yesterday
SELECT percentile(duration, 50, 95, 99) FROM Transaction
WHERE appName = 'checkout-api' SINCE 1 hour ago COMPARE WITH 1 day ago

-- Slowest endpoints
SELECT average(duration), percentile(duration, 95), count(*) FROM Transaction
WHERE appName = 'checkout-api' FACET name SINCE 1 hour ago LIMIT 20

-- Recent error logs
SELECT timestamp, message, level FROM Log
WHERE level = 'ERROR' AND `service.name` = 'checkout-api' SINCE 1 hour ago LIMIT 100
```

For the full syntax reference (event types, functions, `FACET CASES`, subqueries, anti-patterns), read [references/nrql-guide.md](references/nrql-guide.md).

## Safety

Queries run with the signed-in user's permissions. Do not try to work around permission errors. Explain them, and point to the `nrsetup` skill if access looks misconfigured.
