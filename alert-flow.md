# Cost Anomaly Alert Flow

## Detection Pipeline
1. AWS Cost Anomaly Detection ML model analyzes daily spending patterns
2. Model compares current spending against historical baseline
3. When deviation exceeds the learned threshold, an anomaly is created
4. If anomaly impact >= subscription threshold ($10), alert is triggered
5. SNS topic receives the anomaly notification
6. Email is delivered to subscribed addresses

## Sample Anomaly Notification Fields
- **Anomaly ID**: Unique identifier
- **Monitor Name**: Which monitor detected the anomaly
- **Root Cause**: Service and usage type driving the cost increase
- **Impact**: Estimated dollar amount above expected spending
- **Start Date**: When the anomaly began
- **Anomaly Score**: Confidence level of the detection

## Triage Steps
1. Open the Cost Anomaly Detection console
2. Click the anomaly ID to view details
3. Identify the root cause service and usage type
4. Check recent deployments or configuration changes
5. Determine if the spending is legitimate or accidental
6. Take corrective action (terminate resources, revert changes)
7. Document the incident
