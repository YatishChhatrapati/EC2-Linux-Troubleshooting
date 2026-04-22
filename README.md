# AWS EC2 & Linux Troubleshooting Scenarios

Hands-on troubleshooting scenarios practised on AWS Free Tier
using Ubuntu Server — part of my transition into Cloud Support Engineering.

---

## Environment

- Amazon EC2 — Ubuntu Server (Free Tier)
- AWS Security Groups and NACLs
- Amazon CloudWatch
- Ubuntu Linux CLI — SSH, top, ps, kill, tail, grep, systemctl, apt
- AWS Management Console

---

## Scenario 1 — EC2 Instance Unreachable via SSH

**Problem:**
Ubuntu EC2 instance was not reachable via SSH from local machine.

**Investigation Steps:**
1. Checked EC2 instance state in AWS Console — status was Running
2. Verified Public IP was assigned to the instance
3. Checked Security Group inbound rules — port 22 (SSH) was missing
4. Checked NACL rules — inbound and outbound were open
5. Added inbound rule: Type SSH, Port 22, Source My IP

**Resolution:**
Added missing SSH inbound rule in Security Group.
Connected to Ubuntu instance successfully after rule update.

```bash
ssh -i "my-key.pem" ubuntu@<public-ip>
```

**Root Cause:**
Security Group inbound rule for port 22 was not configured during instance launch.

**Key Learning:**
Security Groups are stateful — only inbound rule needed.
Always verify Security Group rules first when Ubuntu EC2 is unreachable.
Ubuntu EC2 default SSH user is "ubuntu" — not "ec2-user".

---

## Scenario 2 — Website Down (Port 80 Not Accessible)

**Problem:**
Apache web server was installed on Ubuntu EC2 but
website was not accessible from browser.

**Investigation Steps:**
1. Confirmed EC2 instance was in Running state
2. SSH into Ubuntu instance
3. Checked Apache service status

```bash
sudo systemctl status apache2
```

4. Apache was inactive — started the service

```bash
sudo systemctl start apache2
sudo systemctl enable apache2
```

5. Checked Security Group inbound rules — port 80 (HTTP) was missing
6. Added inbound rule: Type HTTP, Port 80, Source Anywhere
7. Tested website in browser — accessible

**Apache Installation on Ubuntu:**

```bash
sudo apt update
sudo apt install apache2 -y
sudo systemctl start apache2
sudo systemctl enable apache2
```

**Resolution:**
Started Apache service and added HTTP inbound rule in Security Group.
Website became accessible immediately.

**Root Cause:**
Apache service was not running and HTTP inbound rule
was missing in Security Group.

**Key Learning:**
On Ubuntu EC2, web server package is apache2 — not httpd.
Always check both application status and Security Group rules
when website is down.

---

## Scenario 3 — High CPU Utilization

**Problem:**
Ubuntu EC2 instance CPU was at 95 percent.
Instance response was very slow.
CloudWatch alarm triggered.

**Investigation Steps:**
1. Received CloudWatch CPU alarm notification
2. Checked CloudWatch metrics — confirmed CPU spike
3. SSH into Ubuntu EC2 instance

```bash
ssh -i "my-key.pem" ubuntu@<public-ip>
```

4. Identified process consuming high CPU

```bash
top -c
```

5. Checked process details

```bash
ps aux | grep <process-name>
```

6. Terminated the runaway process

```bash
kill -9 <PID>
```

7. Confirmed CPU dropped to normal

```bash
top -c
```

8. Monitored CloudWatch — CPU normalized within 2 minutes

**Resolution:**
Runaway process identified and terminated via Ubuntu CLI.
CPU utilization normalized. CloudWatch alarm auto-resolved.

**Root Cause:**
Uncontrolled background process was consuming
excessive CPU resources on Ubuntu instance.

**Key Learning:**
top, ps aux, and kill commands work the same on Ubuntu.
CloudWatch alarms are essential for early detection
in cloud support environments.

---

## Scenario 4 — Log Analysis for Fault Diagnosis

**Problem:**
Apache application error reported — needed to identify
root cause from Ubuntu system logs.

**Investigation Steps:**
1. Connected to Ubuntu EC2 via SSH
2. Checked Apache error logs

```bash
sudo tail -f /var/log/apache2/error.log
```

3. Filtered specific error keyword

```bash
sudo grep "ERROR" /var/log/apache2/error.log
```

4. Checked system-level logs

```bash
sudo journalctl -u apache2 --since "1 hour ago"
```

5. Identified misconfigured virtual host entry
6. Fixed the configuration file

```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```

7. Tested configuration before restart

```bash
sudo apache2ctl configtest
```

8. Restarted Apache and verified

```bash
sudo systemctl restart apache2
sudo systemctl status apache2
```

**Resolution:**
Misconfiguration identified via log analysis and fixed.
Apache restarted successfully on Ubuntu.

**Root Cause:**
Incorrect virtual host configuration in Apache config file.

**Key Learning:**
Ubuntu Apache logs are at /var/log/apache2/ — not /var/log/httpd/.
Always run apache2ctl configtest before restarting service.
It catches config errors before they cause downtime.

---

## CloudWatch Alarm Setup

CPU utilization alarm configured on Ubuntu EC2 instance:

- Metric: CPUUtilization
- Threshold: Greater than 80 percent
- Period: 5 minutes
- Action: SNS notification triggered
- Result: Alert received when CPU crossed threshold

---

## Ubuntu vs Amazon Linux — Key Differences

| Task | Ubuntu | Amazon Linux |
|---|---|---|
| Web server | apache2 | httpd |
| Install package | sudo apt install | sudo yum install |
| Log location | /var/log/apache2/ | /var/log/httpd/ |
| Default SSH user | ubuntu | ec2-user |
| Config test | apache2ctl configtest | httpd -t |

---

## Skills Demonstrated

| Skill | Scenario |
|---|---|
| EC2 Ubuntu troubleshooting | 1, 2, 3 |
| Security Group diagnosis | 1, 2 |
| Ubuntu Linux CLI | All |
| Apache2 on Ubuntu | 2, 4 |
| Log analysis — tail, grep, journalctl | 4 |
| CloudWatch monitoring | 3 |
| Process management — top, ps, kill | 3 |
| Root cause analysis (RCA) | All |
| Incident documentation | All |

---

## About

6+ years of Operations & Support experience at Accenture and Genpact
— transitioning into AWS Cloud Support Engineering.
All scenarios practised on AWS Free Tier using Ubuntu Server.
