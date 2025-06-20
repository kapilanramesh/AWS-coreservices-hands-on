
Instead of using a **custom metric**, let’s now build everything using a **default metric** that AWS CloudWatch already collects for EC2 instances.

##### ✅ Use Default EC2 Metrics (No custom push needed)

CloudWatch automatically collects these metrics for your EC2 instance:

| Metric Name         | Description                        |
| ------------------- | ---------------------------------- |
| `CPUUtilization`    | CPU usage percentage               |
| `DiskReadBytes`     | Bytes read from disk               |
| `DiskWriteBytes`    | Bytes written to disk              |
| `NetworkIn` / `Out` | Network traffic (bytes)            |
| `StatusCheckFailed` | If instance or system check failed |
🧠 These come from the EC2 agent **even without** CloudWatch Agent.

##### 🔔 Let’s Now Create Everything Using `CPUUtilization`:

We will:

1. **Create an Alarm** on high CPU
    
2. **Send notification via SNS**
    
3. **Add to Dashboard**
    
4. **(Optionally)** Search log metrics if you enabled agent logs

##### 🧭 Step-by-Step Plan

##### ✅ Step 1: Create an Alarm for High CPU

This alarm triggers when CPU > 70% for 2 minutes.

```
    aws cloudwatch put-metric-alarm \
    --alarm-name HighCPU \
    --metric-name CPUUtilization \
    --namespace AWS/EC2 \
    --statistic Average \
    --period 60 \
    --threshold 70 \
    --comparison-operator GreaterThanThreshold \
    --evaluation-periods 2 \
    --dimensions Name=InstanceId,Value=i-xxxxxxxxxxxxxxxxx \
    --alarm-actions arn:aws:sns:region:account-id:MyAlarmTopic \
    --ok-actions arn:aws:sns:region:account-id:MyAlarmTopic \
    --unit Percent
```

> [!Replace values]
> ✅ Replace:
> 
> - `i-xxxxxxxxxx` with your EC2 instance ID
>     
> - `arn:aws:sns:...` with your actual SNS topic ARN

| **Option**              | **Value**                                   | **Explanation**                                                   |
| ----------------------- | ------------------------------------------- | ----------------------------------------------------------------- |
| `--alarm-name`          | `HighCPU`                                   | Name of the alarm                                                 |
| `--metric-name`         | `CPUUtilization`                            | Metric to monitor                                                 |
| `--namespace`           | `AWS/EC2`                                   | Default namespace for EC2 instance metrics                        |
| `--statistic`           | `Average`                                   | Uses average CPU value per period                                 |
| `--period`              | `60` (seconds)                              | Time window (1 minute) for evaluating metric                      |
| `--threshold`           | `70`                                        | Alarm triggers if average CPU ≥ 70%                               |
| `--comparison-operator` | `GreaterThanThreshold`                      | Triggers when value > threshold                                   |
| `--evaluation-periods`  | `2`                                         | Condition must be met for 2 consecutive periods (2 minutes total) |
| `--dimensions`          | `Name=InstanceId,Value=i-xxxxxxxxxxxxxxxxx` | Filters the metric for a specific EC2 instance ID                 |
| `--alarm-actions`       | `arn:aws:sns:...:MyAlarmTopic`              | SNS topic to notify when alarm goes into ALARM state              |
| `--ok-actions`          | `arn:aws:sns:...:MyAlarmTopic`              | SNS topic to notify when alarm goes back to OK state              |
| `--unit`                | `Percent`                                   | Unit of measurement for this metric                               |

##### ✅ Step 2: Create a Dashboard with CPU Graph

```
aws cloudwatch put-dashboard \
  --dashboard-name "EC2-Monitor" \
  --dashboard-body '{
    "widgets": [
      {
        "type": "metric",
        "x": 0, "y": 0, "width": 12, "height": 6,
        "properties": {
          "metrics": [
            ["AWS/EC2", "CPUUtilization", "InstanceId", "i-xxxxxxxxxxxxxxxxx"]
          ],
          "title": "EC2 CPU Usage",
          "stat": "Average",
          "region": "your-region"
        }
      }
    ]
  }'
```

> [!replace values]
> Replace `"i-xxxxxxxxxxxxxxxxx"` and `"your-region"` with your values.

##### ✅ Step 3: View Everything in CloudWatch Console

- Go to **CloudWatch → Alarms** → See `HighCPU` alarm
    
- Go to **CloudWatch → Dashboards** → See `EC2-Monitor`

##### 📩 Optional: Setup SNS Notification

If not already set up, create an SNS topic and subscribe an email:

```
aws sns create-topic --name MyAlarmTopic

aws sns subscribe \
  --topic-arn arn:aws:sns:region:account-id:MyAlarmTopic \
  --protocol email \
  --notification-endpoint your-email@example.com

```

Then confirm the subscription from your inbox.

##### DELETING COMMANDS

##### ✅ 1. 🗑️ **Delete CloudWatch Alarms**

```
aws cloudwatch delete-alarms --alarm-names "HighCPU" "HighMemory"

```

🔹 You can list multiple alarm names separated by space.  
🔸 To see alarms:

```
aws cloudwatch describe-alarms

```

##### ✅ 2. 🧹 **Delete Dashboards**

```
aws cloudwatch delete-dashboards --dashboard-names "EC2-Monitor" "MyAppDashboard"

```

🔍 To see dashboards:

```
aws cloudwatch list-dashboards

```

##### ✅ 3. ❌ **Delete Custom Log Groups**

```
aws logs delete-log-group --log-group-name "MySyslogGroup"
```

🔍 To see available log groups:

```
aws logs describe-log-groups

```

##### ✅ 4. 🗑️ **Delete Custom Metrics?**

##### 📌 Important Note:

**You CANNOT directly delete metrics in CloudWatch.**  
They expire automatically after **15 months** if **no new data is pushed**.

> So for custom metrics like `MyApp/Errors`, stop pushing data — and it will disappear over time.

##### ✅ 5. 🧽 Clean Up SNS Topics (Alarms Use This)

```
aws sns delete-topic --topic-arn arn:aws:sns:region:account-id:MyAlarmTopic
```

To see topic ARNs:

```
aws sns list-topics
```

##### ✅ Summary of Commands

| What              | Command Example                    |
| ----------------- | ---------------------------------- |
| Delete alarms     | `aws cloudwatch delete-alarms`     |
| Delete dashboards | `aws cloudwatch delete-dashboards` |
| Delete log group  | `aws logs delete-log-group`        |
| Delete SNS topic  | `aws sns delete-topic`             |
| Custom metrics    | ❌ No direct delete (auto-expire)   |