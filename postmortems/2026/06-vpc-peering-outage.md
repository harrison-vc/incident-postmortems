# Postmortem: Inter-Region VPC Peering Outage

**Date**: 2026-04-15  
**Authors**: harrison-vc  
**Status**: Resolved  

## Summary
At 09:00 UTC, traffic between the US-EAST-1 (Prod) and US-WEST-2 (DR) VPCs was interrupted, causing database replication failures and cross-region latency spikes for 45 minutes.

## Impact
- **Service Affected**: Cross-region Database Replication (Aurora Global)
- **Customers Impacted**: Internal reporting systems and DR synchronization.
- **Data Lag**: Replication lag increased to 45 minutes.
- **Connectivity**: 100% packet loss between CIDR blocks 10.0.0.0/16 and 10.1.0.0/16.

## Timeline
- **09:05 UTC**: CloudWatch Alarm `GlobalReplicationLag` triggered.
- **09:10 UTC**: SRE team confirmed instances in US-WEST-2 could not reach US-EAST-1 via private IP.
- **09:15 UTC**: Initial triage ruled out Security Groups (No changes in 48h).
- **09:20 UTC**: Identified the VPC Peering Connection status as `Active`, but `ping` tests failed.
- **09:30 UTC**: Discovered a missing entry in the US-EAST-1 Main Route Table for the peering target.
- **09:40 UTC**: Manually added the route: `10.1.0.0/16 -> pcx-01234567`.
- **09:45 UTC**: Connectivity restored; replication lag began decreasing.

## Detection
The incident was detected via a CloudWatch Alarm monitoring Aurora Global Database replication lag.

## Root Cause
A network maintenance script designed to clean up "orphaned" routes incorrectly flagged the cross-region peering route as unused because it was not explicitly tagged with the `ManagedBy: Terraform` key.

## Contributing Factors
- Lack of resource tagging on legacy VPC route table entries.
- Automated cleanup scripts running with high-privilege `DeleteRoute` permissions without a "dry-run" validation phase.

## Resolution
- **Immediate**: Manually restored the route to the peering connection.
- **Verification**: Verified end-to-end connectivity using `nc -zv` between cross-region application nodes.

## Prevention
- **Action Item**: Implement mandatory tagging for all Route Table entries via AWS Config rules. [Priority: High]
- **Action Item**: Update the cleanup script to include a 24-hour "soft-delete" (tagging for deletion) before actual removal. [Priority: Medium]

## Takeaways
Infrastructure "cleanup" automation must be as robust as deployment automation. Explicit tagging is the only reliable way to prevent automated tools from misidentifying critical resources.
