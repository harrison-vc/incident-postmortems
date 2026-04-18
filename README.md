[![Lint](https://github.com/harrison-vc/incident-postmortems/actions/workflows/lint.yml/badge.svg)](https://github.com/harrison-vc/incident-postmortems/actions/workflows/lint.yml)

# incident-postmortems

A collection of incident postmortems documenting real-world infrastructure and application failures. This repository demonstrates the process of analyzing outages, identifying root causes, and implementing preventative measures to improve system reliability.

## Format

Each postmortem follows a standard, blame-free, and production-oriented structure:

- **Summary**: High-level overview of the incident.
- **Impact**: Quantified effect on customers and systems.
- **Timeline**: Chronological sequence of events from detection to resolution.
- **Detection**: How the incident was identified (e.g., monitor, alert, customer report).
- **Root Cause**: The underlying technical failure mechanism.
- **Contributing Factors**: Environmental or process issues that exacerbated the incident.
- **Resolution**: Steps taken to restore normal service.
- **Prevention**: Action items to avoid recurrence.
- **Takeaways**: Lessons learned for future system design and operational processes.

## Methodology

These reports reflect an engineering culture that values transparency, learning from failure, and building resilient systems through iterative improvement.

## Postmortem Workflow

1. **Initial Draft**: SRE/On-call engineer drafts the postmortem within 24 hours of resolution.
2. **Review**: Peer review for technical accuracy and blame-free tone.
3. **Publication**: Shared internally to engineering teams.
4. **Action Tracking**: Prevention items are moved to the engineering backlog.
