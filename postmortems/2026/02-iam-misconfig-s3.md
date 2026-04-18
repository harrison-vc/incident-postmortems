# Postmortem: IAM Permission Error (S3 Upload)

**Date**: 2026-03-22  
**Authors**: harrison-vc  
**Status**: Resolved  

## Summary
For 12 hours, the `log-aggregator` service on EC2 failed to upload daily logs to the `prod-logs-archive` S3 bucket, resulting in potential data loss.

## Impact
- **Service Affected**: Centralized Logging
- **Data Loss**: No logs lost (Retained on instance), but analysis was delayed by 12 hours.
- **Error Rate**: 100% fail rate for log uploads during the window.

## Timeline
- **08:00 UTC**: Deployment of a new IAM role `LogCollectorV2` via Terraform.
- **08:05 UTC**: First upload failures recorded in application logs.
- **20:00 UTC**: Incident detected during a routine check of the logging dashboard (No alert was configured).
- **20:15 UTC**: Identified the missing object-level ARN in the IAM policy.
- **20:30 UTC**: Policy updated to include `arn:aws:s3:::prod-logs-archive/*`.
- **20:45 UTC**: Service resumed successful uploads.

## Detection
The incident was detected manually by an engineer checking the centralized logs. No automated alerts were in place for this failure mode.

## Root Cause
The new IAM policy resource block only specified the bucket ARN (`arn:aws:s3:::prod-logs-archive`), which grants bucket-level permissions but does not permit `PutObject` operations on the objects within the bucket.

## Contributing Factors
- The IAM policy was manually refactored in Terraform without using a standard template.
- Lack of proactive monitoring for upload failures (Silent failures).

## Resolution
- Updated the IAM policy to grant permissions on the object level (`/*`).

## Prevention
- **Action Item**: Implement a CloudWatch Alarm for "Log Upload Failed" log patterns. [Priority: High]
- **Action Item**: Adopt a standardized Terraform module for all S3-related IAM policies. [Priority: Medium]

## Takeaways
Even minor ARN mismatches can lead to complete service failures. Proactive alerting on failed outbound operations is critical for detecting silent failures.
