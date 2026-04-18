# Postmortem: Security Group Ingress Omission (HTTPS Outage)

**Date**: 2026-04-12  
**Authors**: harrison-vc  
**Status**: Resolved  

## Summary
At 11:30 UTC, a Terraform deployment intended to harden the security posture of the production web tier inadvertently removed the inbound rule for HTTPS (Port 443), causing a 20-minute outage for all external users.

## Impact
- **Service Affected**: Public Website (Frontend)
- **Customers Impacted**: All users visiting the platform via HTTPS.
- **Error Rate**: 100% timeout rate for HTTPS traffic.

## Timeline
- **11:25 UTC**: Terraform apply started for the `web-sg-prod` security group.
- **11:30 UTC**: First external reports of site being down (Connection timed out).
- **11:35 UTC**: Verified the application was still running locally and bound to port 443.
- **11:40 UTC**: Inspected the Security Group rules via AWS Console and identified the missing port 443 rule.
- **11:45 UTC**: Manually patched the Security Group rule. Service restored.
- **12:00 UTC**: Corrected the Terraform code and applied a proper state fix.

## Detection
The incident was detected by external synthetic monitors alerting on site unavailability.

## Root Cause
A refactoring of the Terraform `aws_security_group` resource transitioned from inline `ingress` blocks to standalone `aws_security_group_rule` resources, but the port 443 rule was missed during the migration.

## Contributing Factors
- Lack of a "Plan Review" process for security-related infrastructure changes.
- Manual verification of connectivity was not performed immediately following the `terraform apply`.

## Resolution
- Manually added the rule for immediate restoration.
- Corrected the Terraform state to match the manual fix.

## Prevention
- **Action Item**: Implement `checkov` or a similar policy-as-code tool to scan for mandatory rules during CI. [Priority: High]
- **Action Item**: Automate a connectivity test (Health Check) immediately following infrastructure changes in the pipeline. [Priority: Medium]

## Takeaways
Infrastructure as Code is powerful but can lead to mass failure if rules are accidentally omitted. Policy-as-code validation is a necessary safeguard.
