# Cost Anomaly Response Runbook

## Severity Levels
| Level    | Impact       | Response Time | Escalation         |
|----------|-------------|---------------|---------------------|
| P1       | > $100/day  | 30 minutes    | Team Lead + Manager |
| P2       | $50-100/day | 2 hours       | Team Lead           |
| P3       | $10-50/day  | Next business day | Self-resolve    |

## Step-by-Step Response

### 1. Acknowledge the Alert
- Reply to the SNS notification email
- Post in #cloud-costs Slack channel

### 2. Identify the Root Cause
```bash
# Get recent anomalies
aws ce get-anomalies \
  --date-interval '{"StartDate":"2026-03-22","EndDate":"2026-03-29"}' \
  --query 'Anomalies[].{ID:AnomalyId, Score:AnomalyScore, Impact:Impact.TotalImpact}'

# Check which service spiked
aws ce get-cost-and-usage \
  --time-period Start=2026-03-28,End=2026-03-29 \
  --granularity DAILY \
  --metrics "BlendedCost" \
  --group-by Type=DIMENSION,Key=SERVICE
```

### 3. Investigate Resources
```bash
# List recently launched EC2 instances
aws ec2 describe-instances \
  --filters "Name=launch-time,Values=2026-03-28*" \
  --query 'Reservations[].Instances[].{ID:InstanceId, Type:InstanceType, State:State.Name, LaunchTime:LaunchTime}'

# Check for large S3 operations
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventSource,AttributeValue=s3.amazonaws.com \
  --start-time "2026-03-28T00:00:00Z" \
  --max-results 20
```

### 4. Take Corrective Action
- Terminate unused or accidental resources
- Revert configuration changes that caused the spike
- Apply resource tags for tracking

### 5. Post-Incident Documentation
- Record root cause in the incident log
- Update anomaly threshold if alert was a false positive
- Share findings in the next team standup

## Alert Frequency Decision

For production, **IMMEDIATE** frequency is the right choice for the all-services monitor because:
- Cost spikes can compound quickly — a runaway process left overnight can generate hundreds of dollars in unexpected charges
- Real-time alerts allow same-day triage and corrective action before the anomaly grows
- The $10 threshold filters out noise while still catching meaningful deviations

**DAILY** is appropriate for the EC2 monitor as a secondary digest — it provides a consolidated view without duplicate noise from the IMMEDIATE subscription.
