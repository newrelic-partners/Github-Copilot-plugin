# Basic structure

```sql
SELECT <attributes/functions>
FROM <event_type>
WHERE <conditions>
FACET <grouping>
SINCE <time_range>
LIMIT <count>
```

## Event types

| Event type | Description | Key attributes |
|------------|-------------|----------------|
| Transaction | APM transactions | appName, name, duration, error |
| TransactionError | Application errors | error.class, error.message, transactionName |
| Span | Distributed trace spans | traceId, duration.ms, service.name, category |
| Log | Log events | message, level, entity.guid, service.name |
| Metric | Dimensional metrics | metricName, entity.guid |
| SystemSample | Host metrics | cpuPercent, memoryUsedPercent, hostname |
| ProcessSample | Process metrics | cpuPercent, memoryResidentSizeBytes |
| NetworkSample | Network metrics | receiveBytesPerSecond, transmitBytesPerSecond |
| Deployment | Deployment events | appName, revision, user |

## SELECT functions

```sql
count(*)                          -- count events
average(duration)                 -- mean
sum(duration)
min(duration), max(duration)
percentile(duration, 50, 95, 99)
rate(count(*), 1 minute)          -- events per time unit
uniqueCount(userId)               -- distinct values
latest(attribute)                 -- most recent value
filter(count(*), WHERE error IS true)
percentage(count(*), WHERE error IS true)
histogram(duration, 100, 20)
derivative(average(duration), 1 minute)
```

## WHERE patterns

```sql
WHERE attribute = 'value'
WHERE attribute != 'value'
WHERE duration > 1000
WHERE duration BETWEEN 100 AND 1000
WHERE message LIKE '%timeout%'
WHERE message NOT LIKE '%health%'
WHERE status IN ('error', 'warning')
WHERE error IS true
WHERE attribute IS NULL
WHERE (status = 'error' OR status = 'warning') AND appName = 'MyApp'
```

## FACET

```sql
FACET appName
FACET appName, host
FACET appName LIMIT 20
FACET CASES (
  WHERE duration < 100 AS 'Fast',
  WHERE duration < 1000 AS 'Normal',
  WHERE duration >= 1000 AS 'Slow'
)
FACET buckets(duration, 100, 20)
```

## Time ranges and TIMESERIES

```sql
SINCE 30 minutes ago
SINCE 1 hour ago
SINCE 1 day ago
SINCE 2 hours ago UNTIL 1 hour ago
SINCE today
SINCE '2026-03-01 00:00:00'

TIMESERIES AUTO
TIMESERIES 1 minute
TIMESERIES MAX            -- up to 366 buckets
COMPARE WITH 1 day ago
```

## Subqueries

```sql
SELECT count(*) FROM Transaction
WHERE appName IN (SELECT uniques(appName) FROM Transaction WHERE error IS true)
SINCE 1 hour ago
```

## Anti-patterns

| Avoid | Prefer |
|-------|--------|
| No `SINCE` clause | Always add a time range |
| `SELECT *` without `LIMIT` | Add `LIMIT 100`, or select specific attributes |
| No app/service filter | `WHERE appName = '...'` |
| `LIKE '%error%'` over a day | `level = 'ERROR' AND message LIKE '%timeout%'` over 1 hour |
| Raw rows over 30 days | Aggregate, for example `FACET dateOf(timestamp)` |
| `FACET traceId` | `FACET name LIMIT 20` |

## Time range progression

```
SINCE 30 minutes ago  -> quick check
SINCE 1 hour ago      -> standard investigation
SINCE 4 hours ago     -> pattern analysis
SINCE 24 hours ago    -> historical trends
```
