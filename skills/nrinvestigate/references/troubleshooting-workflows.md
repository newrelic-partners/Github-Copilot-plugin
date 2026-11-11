# Troubleshooting workflows (NRQL)

NRQL sequences for common production scenarios. Run them with `execute_nrql_query`. Replace the example service names.


## Production errors

```sql
-- 1. Error rate
SELECT percentage(count(*), WHERE error IS true) AS 'Error Rate %',
       count(*) AS 'Total Requests',
       filter(count(*), WHERE error IS true) AS 'Errors'
FROM Transaction WHERE appName = 'checkout-api' SINCE 1 hour ago

-- 2. Error types
SELECT count(*) FROM TransactionError WHERE appName = 'checkout-api'
FACET error.class, error.message SINCE 1 hour ago LIMIT 20

-- 3. Error logs
SELECT timestamp, message, level, error.class, error.message FROM Log
WHERE level = 'ERROR' AND `service.name` = 'checkout-api' SINCE 1 hour ago LIMIT 100

-- 4. Failing spans
SELECT traceId, duration.ms, `service.name`, error.message FROM Span
WHERE error IS true AND `service.name` = 'checkout-api' SINCE 1 hour ago LIMIT 20

-- 5. Recent deployments
SELECT timestamp, revision, user, changelog FROM Deployment
WHERE appName = 'checkout-api' SINCE 24 hours ago
```

## Performance degradation

```sql
-- 1. Current vs historical latency
SELECT percentile(duration, 50, 95, 99) FROM Transaction
WHERE appName = 'api-service' SINCE 1 hour ago COMPARE WITH 1 day ago

-- 2. Slow endpoints
SELECT average(duration) AS 'Avg', percentile(duration, 95) AS 'P95', count(*) FROM Transaction
WHERE appName = 'api-service' FACET name SINCE 1 hour ago LIMIT 20

-- 3. Database time
SELECT average(duration.ms) AS 'Avg DB Time', count(*) AS 'Query Count' FROM Span
WHERE category = 'datastore' AND `service.name` = 'api-service'
FACET db.operation, db.collection SINCE 1 hour ago LIMIT 20

-- 4. Host resources
SELECT average(cpuPercent), average(memoryUsedPercent) FROM SystemSample
WHERE hostname LIKE '%api%' FACET hostname TIMESERIES AUTO SINCE 4 hours ago

-- 5. External calls
SELECT average(duration.ms), count(*) FROM Span
WHERE category = 'http' AND `service.name` = 'api-service'
FACET `http.url`, `http.status_code` SINCE 1 hour ago LIMIT 20
```

## Service dependencies

```sql
SELECT count(*), average(duration.ms) FROM Span
WHERE `service.name` = 'frontend' FACET `peer.service`, category SINCE 1 hour ago

SELECT count(*) FROM Span WHERE error IS true
FACET `service.name`, `peer.service` SINCE 1 hour ago
```

## Alert investigation

```sql
SELECT average(duration), percentile(duration, 95, 99), count(*) FROM Transaction
WHERE appName = 'MyApp' TIMESERIES AUTO SINCE 2 hours ago

SELECT count(*) FROM Transaction WHERE appName = 'MyApp' AND error IS true
TIMESERIES AUTO SINCE 2 hours ago

SELECT average(cpuPercent), average(memoryUsedPercent) FROM SystemSample
WHERE hostname IN (SELECT uniques(host) FROM Transaction WHERE appName = 'MyApp')
TIMESERIES AUTO SINCE 2 hours ago

SELECT timestamp, message, level FROM Log
WHERE entity.name = 'MyApp' SINCE 2 hours ago LIMIT 200
```

## Checklist

- Start with 30 to 60 minute windows.
- Always `LIMIT` raw-event queries.
- Scope by `appName`, `service.name`, or entity GUID.
- Check golden signals first, then drill down.
- Check for recent deployments.
