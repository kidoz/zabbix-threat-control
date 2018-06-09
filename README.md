# Zabbix Threat Control
Оur plugin transforms your Zabbix monitoring system into an efficient harvester to manage the vulnerabilities, risk and security of your infrastructure.

## What the plugin does

The plugin provides in Zabbix the information about the vulnerabilities of the entire infrastructure: the scope of the impact, and a list of affected hosts and ways to fix them.

The Information is displayed in zabbix in the following form:

- CVSS score for each server.
- Command to fix all detected vulnerabilities for each server.
- List of security bulletins with the description of the vulnerabilities of the packages for the whole infrastructure.
- List of the packages that are vulnerable for the whole infrastructure.


![](https://github.com/vulnersCom/zabbix-threat-control/blob/master/docs/hosts.gif)


The Information about the security bulletins and packages is presented in a following form:

- Index of the impact on the infrastructure
- CVSS score of package or bulletins.
- Number of affected servers.
- A detailed list of affected hosts.
- Hyperlink to the description of the bulletin.

![](https://github.com/vulnersCom/zabbix-threat-control/blob/master/docs/pkgs.gif)

Sometimes it is quite difficult to update all packages on all servers to a version that fixes vulnerabilities. The offered format of the information representation helps to work selectively with both needed servers and particular packages.

This approach allows you to fix the vulnerabilities of different strategies:

- or all vulnerabilities on certain servers;
- or a specific vulnerability in the entire infrastructure.

This can be done directly from Zabbix (using its standard functionality) by an administrator command or in automatic mode.

## How the plugin works

Using Zabbix API, receives the list of installed packages, the name and version of the OS from all the servers in the infrastructure (if the "Vulners OS-Report" template is linked with them).

Transmits the data to Vulners, and receives information on the vulnerabilities of each server.

Processes the received information, aggregates it and display it in Zabbix.

## Requirements

The plugin requires:

- python 3.6
- python modules: pyzabbix, jpath, requests 
- zabbix-sender utility for sending monitoring data to Zabbix server.

## Installation

### RHEL

rpm -Uhv https://repo.vulners.com/redhat/vulners-repo-2018.06.09.el.noarch.rpm

**On zabbix-server host:**

    yum install zabbix-threat-control-main

**For all servers that require a vulnerability scan:**

    yum install zabbix-threat-control-host


### Debian

    wget https://repo.vulners.com/debian/vulners-repo_2018.06.09+stretch_all.deb
    dpkg -i vulners-repo_2018.06.09+stretch_all.deb

**On zabbix-server host:**

    apt-get update && apt-get install zabbix-threat-control-main

**For all servers that require a vulnerability scan:**

    apt-get update && apt-get install zabbix-threat-control-host

### From source

**On zabbix-server host:**

    git clone https://github.com/vulnersCom/zabbix-threat-control.git
    mkdir -p /opt/monitoring/zabbix-threat-control
    cp zabbix-threat-control/ztc* /opt/monitoring/zabbix-threat-control/
    chown -R zabbix:zabbix /opt/monitoring/zabbix-threat-control
    chmod 640 /opt/monitoring/zabbix-threat-control/ztc_config.py
    touch /var/log/zabbix-threat-control.log
    chown zabbix:zabbix /var/log/zabbix-threat-control.log
    chmod 664 /var/log/zabbix-threat-control.log

**For all servers that require a vulnerability scan:**

    git clone https://github.com/vulnersCom/zabbix-threat-control.git
    mkdir -p /opt/monitoring/
    cp -R zabbix-threat-control/os-report /opt/monitoring/
    chown -R zabbix:zabbix /opt/monitoring/os-report

## Сonfiguration

### Configuration file

Configuration is located in file `/opt/monitoring/zabbix-threat-control/ztc_config.py`

Enter the following in configuration file to connect to Zabbix:
-	The URL, username and password for connection with API. The User should have rights to create groups, hosts and templates in Zabbix.
-	Address and port of the Zabbix-server for pushing data using the zabbix-sender.

Here is example of config file:

```
zbx_pass = 'yourpassword'
zbx_user = 'yourlogin'
zbx_url = 'https://zabbixfront.yourdomain.com'

zbx_server = 'zabbixserver.yourdomain.com'
zbx_port = '10051'
```
### Vulners

Now you should get Vulners api-key. Log in to vulners.com, go to userinfo space https://vulners.com/userinfo. Then you should choose "apikey" section.
Choose "scan" in scope menu and click "Generate new key". You will get an api-key, which looks like this:
**RGB9YPJG7CFAXP35PMDVYFFJPGZ9ZIRO1VGO9K9269B0K86K6XQQQR32O6007NUK**

You wll need to write Vulners api-key into configuration (parameter ```vuln_api_key```).

```
vuln_api_key = 'RGB9YPJG7CFAXP35PMDVYFFJPGZ9ZIRO1VGO9K9269B0K86K6XQQQR32O6007NUK'
```

### Zabbix

1. To create all the necessary objects in Zabbix, run the `/opt/monitoring/zabbix-threat-control/ztc_create.py` script.

This will create these objects in Zabbix using the API:
- A template; through which data will be collected from the servers.
- Zabbix hosts; for obtaining data on vulnerabilities.
- Dashboard; for their display.

2. Following this step. Using the Zabbix web interface, it is necessary to link the "Template Vulners" template to the hosts that you are doing a vulnerabilities scan on.

## Running

Every day at 6 am, Zabbix will automatically receive the name, version and installed packages of the operation system of all servers.
Data processing is performed by script /opt/monitoring/zbx-vulners/zbxvulners.py.
This script is launched by the zabbix-agent every day at 7 am via the item "Service item" on the host "Vulners - Statistics".

## Usage
It will be ready soon...

