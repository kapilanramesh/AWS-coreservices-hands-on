##### 🚀 Amazon CloudWatch Agent Setup on Ubuntu EC2 (with `apt`)

This guide will walk you through installing and configuring the CloudWatch Agent on an **Ubuntu EC2 instance**, and pushing system logs (e.g., `/var/log/syslog`) to **Amazon CloudWatch Logs**.

---

##### ✅ Prerequisites

- Ubuntu EC2 instance (e.g., `t2.micro`)
- IAM Role attached to the instance with:
  - `CloudWatchAgentServerPolicy`
  - (Optional: `AmazonSSMManagedInstanceCore`)

---

##### 📦 Step 1: Download and Install CloudWatch Agent (Debian Package)

```bash
cd ~
wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb

sudo dpkg -i -E ./amazon-cloudwatch-agent.deb
```

---

##### 🛠️ Step 2: Create CloudWatch Agent Configuration File

##### Create Config File:

```bash
sudo nano /opt/aws/amazon-cloudwatch-agent/bin/config.json
```

##### Paste the following configuration:
```json
{
  "agent": {
    "metrics_collection_interval": 60,
    "logfile": "/opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log",
    "run_as_user": "root"
  },
  "metrics": {
    "namespace": "EC2/CustomMetrics",
    "append_dimensions": {
      "InstanceId": "${aws:InstanceId}"
    },
    "metrics_collected": {
      "mem": {
        "measurement": [
          "mem_used_percent"
        ],
        "metrics_collection_interval": 60
      },
      "disk": {
        "measurement": [
          "used_percent"
        ],
        "metrics_collection_interval": 60,
        "resources": [
          "/"
        ]
      }
    }
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/syslog",
            "log_group_name": "MySyslogGroup",
            "log_stream_name": "{instance_id}"
          }
        ]
      }
    }
  }
}
```

Press `Ctrl + O`, `Enter`, then `Ctrl + X` to save and exit.

---

##### 🚀 Step 3: Start and Enable CloudWatch Agent

##### Fetch and apply the configuration:
```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json \
  -s
```

### Start the agent (in case it doesn’t auto-start):
```bash
sudo systemctl start amazon-cloudwatch-agent
```

### Enable agent to run at boot:
```bash
sudo systemctl enable amazon-cloudwatch-agent
```

---

##### 📊 Step 4: Verify Agent Status

##### Check if the agent is active:
```bash
sudo systemctl status amazon-cloudwatch-agent
```

Expected output includes:
```
Active: active (running)
```

---

##### 🔍 Step 5: Confirm Logs in AWS Console

1. Open **CloudWatch → Log Groups**
2. Find and click on `MySyslogGroup`
3. Inside, select the **log stream** (should be your EC2 instance ID)
4. View real-time log entries from `/var/log/syslog`

---

##### 🧪 Step 6: Send a Test Log Message

To manually push a test log:
```bash
sudo logger "🚀 Test log entry for CloudWatch from EC2"
```

Then check CloudWatch → Logs → `MySyslogGroup` → Log Stream for the message.

---

##### 🧠 Summary

You have now:
- Installed the CloudWatch agent on Ubuntu
- Configured it to monitor system metrics and logs
- Sent logs to CloudWatch
- Verified end-to-end setup

---

##### 🟦 Next Steps

You can now:
- Set CloudWatch **alarms** based on log or metric thresholds
- Add more logs (e.g., `/var/log/auth.log`, NGINX logs)
- Push **custom metrics**
- Visualize data using **dashboards**
- Use **Log Insights** to query and analyze logs

---

##### ✅ What Is a Custom Metric?

==A **custom metric** is data **you send manually** (via CLI or code) to CloudWatch. It’s useful when CloudWatch cannot auto-detect certain events, like app-specific errors.==

==You decide:==

- ==What to name it (e.g., `"Errors"`)==
    
- ==What value to assign (e.g., `3` errors)==
    
- ==What category to tag it with (e.g., `Service=Login`)==
    

---

##### 🛠️ Step-by-Step: Push a Custom Metric

> [!IAM Role configs]
> Just make sure your EC2 is attached with IAM role  `cloudwatchFullAccess` , `AdministratorAccess` permission. And configured with awscli.

##### 1. Use This CLI Command

```bash
aws cloudwatch put-metric-data \
  --namespace "MyApp" \
  --metric-name "Errors" \
  --value 3 \
  --dimensions "Service=Login"
```

##### What it does:

|Parameter|Meaning|
|---|---|
|`--namespace`|Groups your custom metrics (like a folder).|
|`--metric-name`|The name of the metric you are tracking (`Errors`).|
|`--value`|The count of events you are reporting now (e.g., 3 errors).|
|`--dimensions`|Tags the metric so you know where it came from (e.g., `Service=Login`).|

This tells CloudWatch: "**Log that 3 errors just occurred in the Login service**."

---

##### ❓ Does This Monitor EC2 Automatically?

**No.**

- CloudWatch does NOT detect or monitor real errors here.
    
- You must tell it: "Hey! I just got 3 errors — store that info."
    

##### So where does the error count come from?

From **your logic**:

- Your app detects 3 errors, or
    
- Your log parser sees 3 error lines, or
    
- A script counts them
    

Then you run the CLI (or use a Python script) to **push the value manually**.

---

##### 💬 Examples

##### Push 1 error:

```bash
aws cloudwatch put-metric-data \
  --namespace "MyApp" \
  --metric-name "Errors" \
  --value 1 \
  --dimensions "Service=Login"
```

##### Push 5 errors for Signup service:

```bash
aws cloudwatch put-metric-data \
  --namespace "MyApp" \
  --metric-name "Errors" \
  --value 5 \
  --dimensions "Service=Signup"
```

---

##### 📊 Where to See This Metric

1. Go to **CloudWatch → Metrics**
    
2. Click **“All Metrics”** → Find namespace `MyApp`
    
3. Select metric `Errors`
    
4. View graph by dimensions like `Service=Login`
    

---

##### 🧠 Summary

|Concept|Clarification|
|---|---|
|Auto detects EC2 errors?|❌ No|
|You define the metric?|✅ Yes|
|You push values manually?|✅ Yes|
|CloudWatch stores it?|✅ Yes|
|Can you set alarms or dashboards on it?|✅ Yes!|

---

🔔 Step-by-Step: Create an Alarm on a Custom Metric

##### ✅ What You Already Have

You’ve been pushing this metric:

```
    aws cloudwatch put-metric-data \
     --namespace "MyApp" \
    --metric-name "Errors" \
    --value 3 \
    --dimensions "Service=Login"
```

So CloudWatch now stores:

- Namespace: `MyApp`
    
- Metric Name: `Errors`
    
- Dimension: `Service=Login`

##### ✅ Step 1: Create an SNS Topic (For Email Notification)

```
    aws sns create-topic --name MyAlarmTopic
```

  It returns a topic ARN. Example:

```
    arn:aws:sns:us-east-1:123456789012:MyAlarmTopic
```

##### ✅ Step 2: Subscribe Your Email to SNS Topic

```
    aws sns subscribe \
    --topic-arn arn:aws:sns:us-east-1:123456789012:MyAlarmTopic \
    --protocol email \
    --notification-endpoint your-email@example.com
```

🔔 Go check your inbox and **confirm the subscription**.

##### ✅ Step 3: Create the Alarm for `Errors >= 3`

```
    aws cloudwatch put-metric-alarm \
    --alarm-name HighErrorCount \
    --metric-name Errors \
    --namespace MyApp \
    --statistic Sum \
    --period 60 \
    --threshold 3 \
    --comparison-operator GreaterThanOrEqualToThreshold \
    --evaluation-periods 1 \
    --dimensions Name=Service,Value=Login \
    --alarm-actions arn:aws:sns:us-east-1:123456789012:MyAlarmTopic \
    --ok-actions arn:aws:sns:us-east-1:123456789012:MyAlarmTopic \
    --actions-enabled

```
##### 💡 What This Does

| Option                   | Meaning                                  |
| ------------------------ | ---------------------------------------- |
| `--alarm-name`           | Name of the alarm                        |
| `--metric-name Errors`   | Watch your custom `Errors` metric        |
| `--namespace MyApp`      | Your custom namespace                    |
| `--statistic Sum`        | Total number of errors                   |
| `--period 60`            | Every 60 seconds                         |
| `--threshold 3`          | Alarm if ≥ 3 errors                      |
| `--evaluation-periods 1` | One 1-minute period is enough to trigger |
| `--alarm-actions`        | Send notification via SNS                |
| `--dimensions`           | Focuses on `Service=Login`               |
##### ✅ Step 4: Test It

Push errors manually to trigger the alarm:

```
    aws cloudwatch put-metric-data \
    --namespace "MyApp" \
    --metric-name "Errors" \
    --value 5 \
    --dimensions "Service=Login"
```

Wait a minute, and the alarm should go to **ALARM state** and send an email.

##### 📬 Bonus: Clear the Alarm (Go back to OK)

Push a lower or zero value:

```
    aws cloudwatch put-metric-data \
    --namespace "MyApp" \
    --metric-name "Errors" \
    --value 0 \
    --dimensions "Service=Login"

```



