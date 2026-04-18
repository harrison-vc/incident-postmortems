# Postmortem: Disk Space Exhaustion (Root Partition)

**Date**: 2026-03-05  
**Authors**: harrison-vc  
**Status**: Resolved  

## Summary
At 04:00 UTC, a batch-processing server in the production environment became unresponsive, leading to failures in scheduled data sync jobs for 3 hours.

## Impact
- **Service Affected**: Data Synchronization (Scheduled)
- **Customers Impacted**: Delayed reporting for approximately 2,000 customers.
- **Error Rate**: 100% fail rate for cron-based jobs on the affected instance.

## Timeline
- **04:05 UTC**: CloudWatch Alarm `DiskSpaceUtilization` triggered (>90%).
- **05:00 UTC**: Alert was noticed by the on-call engineer during a shift change.
- **05:15 UTC**: SSH was unreachable; used AWS SSM Session Manager to gain access.
- **05:30 UTC**: Identified `systemd-journald` as consuming 60GB of space on a 100GB volume.
- **05:45 UTC**: Vacuumed journal logs: `journalctl --vacuum-time=1h`.
- **06:00 UTC**: Recovered 55GB of space. Scheduled jobs resumed.

## Detection
The incident was detected via a CloudWatch Alarm, though response time was delayed due to manual alert review during a shift handover.

## Root Cause
The `systemd-journald` configuration lacked a `SystemMaxUse` limit, allowing logs to grow indefinitely until the root partition was exhausted.

## Contributing Factors
- Recent high-verbosity debug logging was enabled for a specific application but never disabled.
- Default journal configuration was not overridden in the base AMI.

## Resolution
- Emergency cleanup via `journalctl --vacuum-time`.
- Corrected `/etc/systemd/journald.conf` with a `SystemMaxUse=1G` limit.

## Prevention
- **Action Item**: Standardize `journald` configuration across all AMIs. [Priority: High]
- **Action Item**: Integrate disk utilization alerts into PagerDuty for immediate notification. [Priority: Medium]
- **Action Item**: Implement log rotation for all custom application logs. [Priority: High]

## Takeaways
Disk exhaustion is a common and preventable failure. Ensuring proper log rotation and capping journal size are essential baseline configurations for any production Linux environment.
