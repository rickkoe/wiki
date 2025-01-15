---
title: FlashSystem IP Quorum
description: 
published: true
date: 2025-01-15T17:50:58.254Z
tags: flashsystem, ha, quorum
editor: markdown
dateCreated: 2024-11-05T22:02:51.636Z
---

# Install as a Service on RHEL

## *Prerequisites*

-   Verify java is installed

```plaintext
rpm -qa |grep java
which java
# should return --> /usr/bin/java
```

-   Install java if needed

```plaintext
dnf install java
```

## Setup

-   Create a dedicated ipquorum user for starting the service

```plaintext
sudo groupadd ipquorum
sudo useradd -m -s /sbin/nologin -g ipquorum ipquorum
```

-   Create required directories and set permissions for the log file

```plaintext
sudo mkdir -p /opt/ibm/fs5200_ip_quorum/log
sudo chown ipquorum:ipquorum /opt/ibm/fs5200_ip_quorum/log
```

-   Copy the ip*quorum.jar file to /opt/ibm/ip\_*quorum

```plaintext
scp ipquorum.jar root@ibmdev01.esilabs.com:/opt/ibm/ip_quorum
```

-   Create the ipquorum.service file and open in vi

```plaintext
sudo vi /etc/systemd/system/fs5200_ipquorum.service
```

-   Add the following contents to the file in vi

```plaintext
[Unit]
Description=IBM IP Quorum
Documentation=https://www.ibm.com/docs/en/flashsystem-9x00
After=local-fs.target network-online.target
[Service]
Type=simple
WorkingDirectory=/opt/ibm/ip_quorum/log
ExecStart=/usr/bin/java -jar /opt/ibm/ip_quorum/ip_quorum.jar
Restart=on-failure
InaccessibleDirectories=/home /root /var /boot
ReadOnlyDirectories=/etc /usr
User=ipquorum
Group=ipquorum
[Install]
WantedBy=multi-user.target
```

-   Enable the service to start when the servier boots

```plaintext
sudo systemctl enable fs5200_ipquorum
```

-   Start the service

```plaintext
sudo systemctl start fs5200_ipquorum
```

-   Verify the service is running

```plaintext
sudo systemctl status fs5200_ipquorum
```

-   Status should  show active (running).

Example

```plaintext
[root@ibmdev01 ip_quorum]# sudo systemctl status ipquorum
● ipquorum.service - IBM IP Quorum
  Loaded: loaded (/etc/systemd/system/ipquorum.service; enabled; vendor preset: disabled)
  Active: active (running) since Tue 2024-11-05 16:23:38 CST; 2s ago
    Docs: https://www.ibm.com/docs/en/flashsystem-9x00
Main PID: 419939 (java)
   Tasks: 13 (limit: 23204)
  Memory: 55.1M
  CGroup: /system.slice/ipquorum.service
          └─419939 /usr/bin/java -jar /opt/ibm/ip_quorum/ip_quorum.jar

Nov 05 16:23:39 ibmdev01.esilabs.com java[419939]: Successfully parsed the configuration, found 2 nodes.
Nov 05 16:23:39 ibmdev01.esilabs.com java[419939]: Trying to open socket
Nov 05 16:23:39 ibmdev01.esilabs.com java[419939]: Handshaking
Nov 05 16:23:39 ibmdev01.esilabs.com java[419939]: Trying to open socket
Nov 05 16:23:39 ibmdev01.esilabs.com java[419939]: Handshaking
Nov 05 16:23:40 ibmdev01.esilabs.com java[419939]: Creating UID
Nov 05 16:23:40 ibmdev01.esilabs.com java[419939]: Waiting for UID
Nov 05 16:23:40 ibmdev01.esilabs.com java[419939]: *Connecting
Nov 05 16:23:40 ibmdev01.esilabs.com java[419939]: Connected to 172.22.0.61
Nov 05 16:23:40 ibmdev01.esilabs.com java[419939]: Connected to 172.22.0.62
```

-   Storage GUI should show the server listed in IP Quorum