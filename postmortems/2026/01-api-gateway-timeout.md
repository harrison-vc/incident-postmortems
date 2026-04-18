# Postmortem: API Gateway 504 Timeouts (US-EAST-1)

**Date**: 2026-04-10  
**Authors**: harrison-vc  
**Status**: Resolved  

## Summary
Between 14:00 and 15:15 UTC, the external API gateway experienced high latency, resulting in HTTP 504 timeouts for approximately 15% of all client requests.

## Impact
- **Service Affected**: Public API (`/v1/process-data`)
- **Customers Impacted**: ~5,000 unique clients.
- **Error Rate**: Peak 504 rate reached 15%.
- **Latency**: P99 increased from 200ms to 32s.

## Timeline
- **14:05 UTC**: First 504 alert triggered in CloudWatch.
- **14:10 UTC**: SRE team began investigation. Confirmed backend instances are healthy.
- **14:20 UTC**: Identified upstream Payment Gateway as the bottleneck.
- **14:35 UTC**: Increased internal HTTP client timeout from 30s to 45s as a temporary mitigation.
- **14:45 UTC**: Implemented Circuit Breaker to fail-fast on payment requests.
- **15:15 UTC**: Upstream provider resolved their issue. Service returned to baseline.

## Detection
The incident was detected via a CloudWatch Alarm on `HTTPCode_ELB_504_Count` exceeding the threshold of 5 per minute.

## Root Cause
A regional service degradation in the third-party Payment Gateway provider led to processing delays exceeding our internal 30-second timeout.

## Contributing Factors
- Lack of a Circuit Breaker pattern allowed slow upstream responses to tie up backend worker threads.
- Inadequate visibility into specific upstream latency metrics delayed identification of the bottleneck.

## Resolution
- Temporary: Increased internal application timeouts and manually scaled the API tier.
- Permanent: Upstream resolved the regional performance issue.

## Prevention
- **Action Item**: Implement a Circuit Breaker (Hystrix/Resilience4j) for all upstream dependencies. [Priority: High]
- **Action Item**: Add specific P99 latency monitoring for each external API call. [Priority: Medium]

## Takeaways
Synchronous dependencies on third-party APIs are a major point of failure. Moving to an asynchronous processing model for payment status would have mitigated this impact.
