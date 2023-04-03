---
description: >-
  Understand how SIEM works and get comfortable creating simple and advanced
  search queries to look for specific answers from the ingested logs.
---

# Security Information and Event Management

{% hint style="info" %}
SIEM: It is a tool that collects data from various endpoints/network devices across the network, stores them at a centralized place, and performs correlation on them
{% endhint %}

### Network log sources:

1. **Host-Centric Log Sources**

* A user accessing a file
* A user attempting to authenticate.
* A process Execution Activity
* A process adding/editing/deleting a registry key or value.
* PowerShell execution

2. **Network-Centric Log Sources**

* SSH connection
* A file being accessed via FTP
* Web traffic
* A user accessing company's resources through VPN.
* Network file sharing Activity

### Log Sources

#### 1. Windows

Windows records every event that can be viewed through the **Event Viewer** utility

#### 2. Linux

Some of the common locations where Linux store logs are:

* **/var/log/httpd** : Contains HTTP Request  / Response and error logs.
* **/var/log/cron** : Events related to cron jobs are stored in this location.
* /var/log/auth.log and /var/log/secure : Stores authentication related logs.
* /var/log/kern : This file stores kernel related events.

#### 3. Web server

In Linux, common locations to write all apache related logs are **/var/log/apache** or **/var/log/httpd**.
