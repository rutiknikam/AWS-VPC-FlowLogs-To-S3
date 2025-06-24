# 🚀 Configure VPC Flow Logs and Store Logs in S3 Using IAM Role

---

## 📘 Project Overview

This project demonstrates how to enable **VPC Flow Logs** to monitor network traffic inside an AWS VPC. These logs are stored in an **Amazon S3 bucket** for analysis, auditing, and long-term storage. To allow logging access, an **IAM role** is created and used by the VPC Flow Logs service.

---

## 🎯 Objective

* Capture all traffic (Accepted/Rejected) inside a VPC
* Deliver the logs to an S3 bucket securely
* Use IAM Role to manage permissions for log delivery

---

## 🧰 Services Used

* Amazon VPC
* Amazon S3
* IAM (Role + Policy)
* EC2 (for traffic testing)

---

## ⚙️ Requirements

* One VPC (existing or new)
* One public subnet with internet access
* One EC2 instance (to generate traffic)
* One S3 bucket for logs
* One IAM role with permission to write logs to S3

---

## 🧱 Workflow

```
[EC2 Instance] --> [VPC Flow Logs] --> [IAM Role] --> [S3 Bucket]
                        |
                      [All Traffic]
```

---

## 𞯣 Steps to Follow

### ✅ Step 1: Create or Choose a VPC

* Use an existing VPC or create a new one
* Note the **VPC ID**

### ✅ Step 2: Create a Public Subnet

* CIDR Block: must be inside VPC's CIDR (e.g., `10.0.1.0/24`)
* Associate the subnet with a **Route Table** that has a route to the **Internet Gateway**

### ✅ Step 3: Create and Attach Internet Gateway

* Go to **Internet Gateways** → Create one
* Attach it to your VPC
* Edit your subnet's Route Table → Add route:

  * Destination: `0.0.0.0/0`
  * Target: your Internet Gateway

### ✅ Step 4: Launch an EC2 Instance

* Use the public subnet
* Enable **Auto-assign Public IP**
* Name: `vpc-log-test-instance`
* Key Pair: create or select an existing one
* AMI: Amazon Linux 2
* Connect using EC2 Instance Connect
* Run traffic:

  ```bash
  ping google.com
  curl amazon.com
  ```

### ✅ Step 5: Create an S3 Bucket

* Name: `vpc-flow-logs-bucket`
* Region: same as your VPC
* Keep default settings

### ✅ Step 6: Add S3 Bucket Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "vpc-flow-logs.amazonaws.com"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::vpc-flow-logs-bucket/AWSLogs/YOUR-AWS-ACCOUNT-ID/*",
      "Condition": {
        "StringEquals": {
          "aws:SourceAccount": "YOUR-AWS-ACCOUNT-ID"
        }
      }
    }
  ]
}
```

* Replace `YOUR-AWS-ACCOUNT-ID` with your actual ID

### ✅ Step 7: Create IAM Role for Flow Logs

1. Go to IAM → Roles → Create role
2. Use case: **VPC Flow Logs**
3. Attach this inline policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::vpc-flow-logs-bucket/AWSLogs/YOUR-AWS-ACCOUNT-ID/*"
    }
  ]
}
```

4. Trust policy (auto-created):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "vpc-flow-logs.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

### ✅ Step 8: Enable VPC Flow Logs

* Go to VPC → Select your VPC → Flow Logs tab → Create Flow Log
* Filter: All
* Destination: S3
* S3 ARN: `arn:aws:s3:::vpc-flow-logs-bucket`
* IAM Role: Select the one you created
* Click **Create**

---

## 🔍 Sample Log Line Explained

```text
2 123456789012 eni-1234abcd 10.0.1.5 172.217.160.78 443 49152 6 10 840 1687597622 1687597682 ACCEPT OK
```

| Field            | Meaning                  |
| ---------------- | ------------------------ |
| `10.0.1.5`       | Source IP (your EC2)     |
| `172.217.160.78` | Destination IP (Google)  |
| `443`            | Port (HTTPS)             |
| `ACCEPT`         | Action (traffic allowed) |
| `OK`             | Log status               |

---

## 📦 Deliverables

* ✅ IAM Role policy (JSON)
* ✅ S3 Bucket policy (JSON or screenshot)
* ✅ VPC Flow Log setup screenshot
* ✅ S3 log files screenshot
* ✅ Sample log line with explanation
* ✅ This `README.md` file

---

## 📌 Traffic Type Used

* **All Traffic** (`All` filter)
  → To monitor everything: Accept and Reject traffic

---

## 🧠 Why This Setup is Important

* Helps in **network visibility** and **security audits**
* Logs are saved in **S3 for long-term storage**
* IAM Role ensures **secure access control**
* Real traffic tested with EC2 for log verification

---

## ✅ Done!

You now have a working setup that logs all VPC network traffic into S3 using IAM roles 🚀
