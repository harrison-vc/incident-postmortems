# Postmortem: Database Connection Exhaustion (RDS)

**Date**: 2026-02-15  
**Authors**: harrison-vc  
**Status**: Resolved  

## Summary
At 09:30 UTC, the `inventory-service` began returning HTTP 500 errors. Investigation revealed that the RDS PostgreSQL instance was rejecting new connections, causing a complete service outage for 45 minutes.

## Impact
- **Service Affected**: Inventory Management (Write operations)
- **Customers Impacted**: ~12,000 active users.
- **Error Rate**: 100% fail rate for database-dependent requests.

## Timeline
- **09:35 UTC**: PagerDuty alert triggered for `DatabaseConnections` exceeding the threshold of 80% on RDS.
- **09:40 UTC**: 500 errors spiked in the application logs.
- **09:50 UTC**: Verified that `max_connections` limit of 200 was reached.
- **10:00 UTC**: Identified a leak in the connection pool in the recent deployment of `v1.2.4`.
- **10:10 UTC**: Reverted the deployment to `v1.2.3`.
- **10:15 UTC**: Force-killed stale connections via RDS CLI. Service restored.

## Detection
The incident was detected via a CloudWatch Alarm monitoring RDS `DatabaseConnections`.

## Root Cause
A code change in the connection management layer of the application was missing the `db.close()` call in an exception handling block, leading to orphaned connections accumulating rapidly under high load.

## Contributing Factors
- Connection pooling library was misconfigured with no `idleTimeout`.
- Lack of database-side monitoring for specific long-running or orphaned queries.

## Resolution
- Reverted to the previous stable version.
- Manually purged stale backend processes on the RDS instance.

## Prevention
- **Action Item**: Implement a strict `try-finally` or `using` block for all database connection calls. [Priority: High]
- **Action Item**: Configure RDS `idle_in_transaction_session_timeout` to auto-terminate orphaned sessions. [Priority: Medium]

## Takeaways
Connection leaks are a classic failure mode. Using a robust pooling library and configuring server-side timeouts provides multiple layers of protection against application-level leaks.
