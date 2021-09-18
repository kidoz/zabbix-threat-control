#### :bangbang: Updated Zabbix Threat Control to version 1.3.4 :bangbang:
💥 Update breaks the plugin's normal operation!</br>
To make it work, please read the [Update instructions](https://github.com/vulnersCom/zabbix-threat-control/issues/16).</br>
And there's live-chat in Telegram, for technical support use our Telegram live-chat: [@ztcsupport](https://t.me/ztcsupport)

----

# Zabbix Threat Control

Оur plugin transforms your Zabbix monitoring system into vulnerability, risk and security managment system for your infrastructure.

  * [What the plugin does](#what-the-plugin-does)
  * [How the plugin works](#how-the-plugin-works)
  * [Requirements](#requirements)
  * [Installation](#installation)
  * [Сonfiguration](#configuration)
  * [Execution](#execution)
  * [Usage](#usage)
  
## What the plugin does

It provides Zabbix with information about vulnerabilities existing in your entire infrastructure and suggests easily applicable remediation plans.

![](https://github.com/vulnersCom/zabbix-threat-control/blob/master/media/dashboard.png)

Information is displayed in Zabbix in the following format:

- Maximum CVSS score for each server.
- Command for fixing all detected vulnerabilities for each server.
- List of security bulletins with descriptions for vulnerable packages valid for your infrastructure.
- List of all vulnerable packages in your infrastructure.


![](https://github.com/vulnersCom/zabbix-threat-control/blob/master/media/hosts.gif)


Security bulletins and packages information includes:

- Impact index for the infrastructure.
- CVSS score of a package or a bulletin.
- Number of affected servers.
- A detailed list of affected hosts.
- Hyperlink to the description of a bulletin.

![](https://github.com/vulnersCom/zabbix-threat-control/blob/master/media/packages.gif)

Sometimes it is impossible to update all packages on all servers to a version that fixes existing vulnerabilities. The proposed representation permits you to selectively update servers or packages.

This approach allows one to fix vulnerabilities using different strategies:

- all vulnerabilities on a specific server;
- a single vulnerability in the entire infrastructure.

This can be done directly from Zabbix (using its standard functionality) either on the administrator command or automatically.

## How the plugin works

- Using Zabbix API, the plugin receives lists of installed packages, names and versions of the OS from all the servers in the infrastructure (if the "Vulners OS-Report" template is linked with them).
- Transmits the data to Vulners
- Receives information on the vulnerabilities for each server.
- Processes the received information, aggregates it and sends it back to Zabbix via zabbix-sender.
- Finally the result is displayed in Zabbix.

## Requirements

**On zabbix-server host:**

- python 3 (only for ztc scripts)
- python modules: pyzabbix, jpath, requests, vulners
- zabbix version 3.4 is required to create a custom dashboard and a custom polling schedule.
- zabbix-sender utility for sending data to zabbix-server.
- zabbix-get utility for sending a command to fix vulnerabilities on the server.

**On all the servers that require a vulnerability scan:**

- zabbix-agent for collect data and run scripts.

## Installation

### RHEL, CentOS and other RPM-based

    rpm -Uhv https://repo.vulners.com/redhat/vulners-repo.rpm

**On zabbix-server host:**

    yum install zabbix-threat-control-main zabbix-threat-control-host

**On all the servers that require a vulnerability scan:**

    yum install zabbix-threat-control-host


### Debian and other debian-based

    wget https://repo.vulners.com/debian/vulners-repo.deb
    dpkg -i vulners-repo.deb

**On zabbix-server host:**

    apt-get update && apt-get install zabbix-threat-control-main zabbix-threat-control-host

**On all the servers that require a vulnerability scan:**

    apt-get update && apt-get install zabbix-threat-control-host

### From source

**On zabbix-server host:**

    git clone https://github.com/vulnersCom/zabbix-threat-control.git
    mkdir -p /opt/monitoring/zabbix-threat-control
    cp zabbix-threat-control/*.py /opt/monitoring/zabbix-threat-control/
    cp zabbix-threat-control/*.conf /opt/monitoring/zabbix-threat-control/
    chown -R zabbix:zabbix /opt/monitoring/zabbix-threat-control
    chmod 640 /opt/monitoring/zabbix-threat-control/*.conf
    touch /var/log/zabbix-threat-control.log
    chown zabbix:zabbix /var/log/zabbix-threat-control.log
    chmod 664 /var/log/zabbix-threat-control.log

**On all the servers that require a vulnerability scan:**

    git clone https://github.com/vulnersCom/zabbix-threat-control.git
    mkdir -p /opt/monitoring/
    cp -R zabbix-threat-control/os-report /opt/monitoring/
    chown -R zabbix:zabbix /opt/monitoring/os-report

## Configuration

The configuration file is located here: `/opt/monitoring/zabbix-threat-control/ztc.conf`

### Vulners credentials

To use Vulners API you need an api-key. To get it follow the steps bellow:
- Log in to vulners.com.
- Navigate to the userinfo space https://vulners.com/userinfo.
- Choose the "API KEYS" section.
- Select "scan" in the scope menu and click "Generate a new key".
- You will get an api-key, which looks like this:
**RGB9YPJG7CFAXP35PMDVYFFJPGZ9ZIRO1VGO9K9269B0K86K6XQQQR32O6007NUK**

Now you need to add the Vulners api-key into your configuration file (parameter ```VulnersApiKey```).

```
VulnersApiKey = RGB9YPJG7CFAXP35PMDVYFFJPGZ9ZIRO1VGO9K9269B0K86K6XQQQR32O6007NUK
```


### Zabbix credentials

In order to connect to Zabbix you need to specify the following in the configuration file:
-	The URL, username and password. Note that the User should have rights to create groups, hosts and templates in Zabbix.
-	Domain name and port of the Zabbix-server for pushing data using the zabbix-sender.

Here is an example of a valid config file:

```
ZabbixApiUser = yourlogin
ZabbixApiPassword = yourpassword
ZabbixFrontUrl = https://zabbixfront.yourdomain.com

ZabbixServerFQDN = zabbixserver.yourdomain.com
ZabbixServerPort = 10051
```

### Zabbix entity

1. To create all the necessary objects in Zabbix, run the `prepare.py` script with parameters.</br>
`/opt/monitoring/zabbix-threat-control/prepare.py -uvtda`</br>It will verify that zabbix-agent and zabbix-get utilities are configured correctly and create the following objects using Zabbix API:
   * **A template** used to collect data from servers.
   * **Zabbix hosts** for obtaining data on vulnerabilities.
   * **An action** to run the command fixes the vulnerability.
   * **A dashboard** for displaying results.
2. While using the Zabbix web interface, it is necessary to link the "Vulners OS-Report" template with the hosts that you are doing a vulnerabilities scan on.

### Servers that require a vulnerability scan

Zabbix-agent must be able to execute remote commands. For this, change the parameters in the zabbix-agent configuration file `/etc/zabbix/zabbix_agentd.conf`:

```
EnableRemoteCommands=1
LogRemoteCommands=1
``` 

Zabbix-agent must be able to update packages as root. For this, add a line to the file `/etc/sudoers`:

```
zabbix ALL=(ALL) NOPASSWD: /usr/bin/yum -y update *
zabbix ALL=(ALL) NOPASSWD: /usr/bin/apt-get --assume-yes install --only-upgrade *
```

## Execution

- `/opt/monitoring/os-report/report.py`<br />
  Transfers the name, version and installed packages of the operating system to Zabbix.<br />
  Runs with zabbix-agent on all hosts to which the template "Vulners OS-Report" is linked.

- `/opt/monitoring/zabbix-threat-control/scan.py`<br />
  Processes raw data from zabbix and vulners and push them to the monitoring system using zabbix-sender.<br />
  Runs with zabbix-agent on the Zabbix server via the item "Service item" on the host "Vulners - Statistics".

The above scripts are run once a day. The start-up time is selected randomly during the installation and does not change during operation.
  
- `/opt/monitoring/zabbix-threat-control/fix.py`<br />
  Runs commands to fix vulnerabilities on servers. It's executed as a remote command in the action "Vunlers" in Zabbix. 
   

## Usage
It will be ready soon...

