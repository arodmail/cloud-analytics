# Data Acquisition

## Protocol Support

The Cloud Analytics architecture is open to acquiring data through multiple protocols. The following table summarizes protocol support by the architecture. For a protocol to be supported, it must be encrypted and authenticated. 

| | Protocol | Port(s) | Encrypted | Authentication | Supported
|-----|----- |----- |-----|----- |-----
| 1 | HTTPS | *443 | Yes | Yes | Yes
| 2 | SNMPv3 | 161 | Yes | Yes | Yes
| 3 | SSH | 22 | Yes | Yes | Yes
| 4 | SNMPv1/2 | 161 | No | No | No

Note: See [Security Concerns](developer/data-acquisition?id=security-concerns) below for other reasons SNMPv1/2 is not supported. Also see [RAID Log (ART 0350)](architecture/raid) for High Risk identification of SNMP use for device monitoring purposes. 

## 1. HTTPS

HTTPS (HTTP over TLS), is widely supported for its use of an added encryption layer of SSL/TLS to protect web traffic. This support includes encryption of a request URL, query parameters, HTTP headers, and cookies, which may be used to identify a user. HTTPS increases the security posture of the solution through:

- Protection against man-in-the-middle attacks
- Protection of communication against eavesdropping and tampering
- Authentication requires a trusted third party to sign server-side digital certificates
- Ensures the privacy of communication, identity, and transmission of device statistics

HTTPS is especially important in a Zero-Trust environment. Zero-Trust assurances are often required by CIOs and CISOs. 

The primary mechanism by which Cloud Analytics integrates the HTTPS protocol for data acquisition purposes is through a **sensor**: [Prometheus Exporter](https://prometheus.io/docs/instrumenting/exporters/). 

A Prometheus Exporter is a sensor that:

- Fetches statistics from a system
- Turns statistics into Prometheus metrics
- Opens a secure web port that exposes a `/metrics` HTTPS URL to return system metrics
- Exports statistics in a Prometheus compatible format
- Based upon a “pull” and not a “push” model

<img src="_media/node-exporter-model.png" alt="Node Exporter Model" style="display: block; margin-left: auto; margin-right: auto; height: 50%; width: 50%; padding-top: 25px; padding-bottom: 25px;"/>

1. *Endpoints:* span a very broad range of hardware and software across a wide spectrum of device types. 
2. *Time-series Database:* Prometheus is a free, OSS time-series database developed as an alternative to StatsD and Graphite based solutions. Built to provide a multi-dimensional data model, operational simplicity, scalable data collection, and a powerful query language, PromQL.

## 2. SNMPv3

### SNMP Considerations

Simple Network Management Protocol (SNMP), widely used for network monitoring, exposes data in the form of variables in a MIB (Management Information Base) on systems where SNMP is running which can be queried remotely.

### SNMP Exporter

To collect information from SNMP for use by the Prometheus monitoring system, a Prometheus SNMP Exporter exists for this purpose:

https://github.com/prometheus/snmp_exporter

This exporter can be installed in the same manner that the Node Exporter is installed and configured on a monitored device. 

### SNMP Exporter Proxy

In some cases, a Node Exporter cannot be installed on the device to monitor. Reasons for this limitation  include:

- Limited access to a device shell, no root access or sudoers for the Deployment Engineering team to perform the installation
- Restrictions placed by the software vendor on third-party agents co-hosted on the device
- Vendor might cancel support for the software in cases where a client bypasses restrictions and installs a third-party agent on the host

For example, similar restrictions are known to be in place for Guardium devices hosting the Database Activity Monitoring software. To extend the solution in these cases, use of the SNMP Exporter is an option. 

<img src="_media/snmp-proxy-model.png" alt="SNMP Proxy" style="display: block; margin-left: auto; margin-right: auto; height: 70%; width: 70%; padding-top: 25px; padding-bottom: 25px;"/>

1. <b>Endpoint:</b> A host running an SNMP daemon/agent. 
2. <b>Time-series Database:</b> Prometheus is a free, OSS time-series database
3. <b>SNMP Proxy:</b> An instance of [Prometheus SNMP Exporter](https://github.com/prometheus/snmp_exporter) that exposes information gathered from SNMP for use by the Prometheus monitoring system deployed as a containerized application. Deployed in this way, the Prometheus SNMP Exporter becomes a proxy to the managed device for Prometheus to scrape the metrics it exposes. The Prometheus SNMP Exporter as a proxy communicates with the managed device over port 161 to send SNMP requests for data. 

In this model, SNMP proxy leverages out-of-box monitoring services that are expected to be enabled and running on the device, which is the case for most RHEL and other Linux systems. 

## 3. SSH

SSH provides a secure channel over a network through a client–server architecture that connects an SSH client to an SSH server. Encryption used by SSH provides confidentiality and ensures integrity of data transmitted over an unsecured network. 

Often used with automatically generated public-private key pairs, SSH will encrypt a network connection, and then use password authentication on a remote device to log on. Manually generated public-private key pair can be used for authentication to allow a log in without a password.

SSH uses public-key cryptography to authenticate a remote client and allow it to authenticate a user. 

The list of authorized public keys is typically stored in the home directory of the user that is allowed to log in remotely, in the file `~/.ssh/authorized_keys`. 

SSH increases the security posture of the solution through:

- A transport layer that sets up encryption, compression and integrity verification
- For automated remote monitoring and management of servers 
- Support for password authentication and public key-based authentication

<img src="_media/ssh-model.png" alt="SSH Model" style="display: block; margin-left: auto; margin-right: auto; height: 70%; width: 70%; padding-top: 25px; padding-bottom: 25px;"/>

1. <b>Endpoint:</b> A host running an SSH daemon. 
2. <b>Time-series Database:</b> Prometheus is a free, OSS time-series database
3. <b>SSH Exporter:</b> An instance of [Prometheus SSH Exporter](https://github.com/treydock/ssh_exporter) that exposes information gathered from SSH for use by the Prometheus monitoring system deployed as a containerized application. Deployed in this way, the Prometheus SSH Exporter becomes a proxy to the managed device for Prometheus to scrape the metrics it exposes. The Prometheus SSH Exporter as a proxy makes an SSH connection to a remote system over port 22. 

## SSH vs. SNMP

In terms of data collection and insight, SNMP cannot penetrate beyond the surface of the devices being monitored. While traditionally supported by firewalls and routers, SNMP only provides a small subset of data about a device.  

By contrast, SSH allows access and can extract a wide range of configuration data in devices in real time, as a more robust protocol to use for monitoring. SSH can simulate human-like behavior and reach a network far beyond what a standard, SNMP based network monitoring tool can do. 

Using SSH for monitoring a network becomes monitoring in reverse: instead of looking for symptoms (which are the end result of a problem), SSH commands can be designed to look for the possible root causes which provides deeper insight a means to accelerate problem resolution. 

SNMP is only used for status and performance information collection using a network management system, while SSH can be used for interactive troubleshooting and configuration of a device.

### Guardium Sample SNMP Queries

Disk space available in / directory:

```
> snmpget -v 2c -c guardiumsnmp 206.253.228.150 UCD-SNMP-MIB::dskAvail.1
UCD-SNMP-MIB::dskAvail.1 = INTEGER: 18300128
```

Disk space used in / directory:

```
> snmpget -v 2c -c guardiumsnmp 206.253.228.150 UCD-SNMP-MIB::dskUsed.1
UCD-SNMP-MIB::dskUsed.1 = INTEGER: 6044676
````

Available memory:

```
> snmpwalk -v 2c -c guardiumsnmp 206.253.228.150 HOST-RESOURCES-MIB::hrStorageSize.1
HOST-RESOURCES-MIB::hrStorageSize.1 = INTEGER: 24523076
```

Memory Used:

```
> snmpwalk -v 2c -c guardiumsnmp 206.253.228.150 HOST-RESOURCES-MIB::hrStorageUsed.1
HOST-RESOURCES-MIB::hrStorageUsed.1 = INTEGER: 23816600
```

## Security Concerns

A large number of SNMP implementations carry very high risk vulnerabilities as SNMP exposes detailed information in clear text about mission critical assets on a network infrastructure to unauthorized individuals. 

From O'Reilly [Essential SNMP](https://docstore.mik.ua/orelly/networking_2ndEd/snmp/ch07_02.htm):

"Security issues with SNMPv1 and SNMPv2: The biggest problem, of course, is that the read-only and read-write community strings are sent as clear-text strings; the agent or the NMS performs no encryption. Therefore, the community strings are available to anyone with access to a packet sniffer. That certainly means almost anyone on your network with a PC and the ability to download widely available software."

"Solution ... is to prevent the SNMP packets from being visible on your external network connections and parts of your network where you don't want them to appear. This requires configuring your routers and firewalls with access lists that block SNMP packets from the outside world (which may include parts of your own network)."

"If you don't trust the users of your network, you may want to set up a separate administrative network to be used for SNMP queries and other management operations. This is expensive and inflexible -- it's hard to imagine extending such a network beyond your core routers and servers -- but it may be what your situation requires."

### List of CVE Security Vulnerabilities for SNMP Version 1 and 2

|CVE ID | Gained Access Level | Complexity
|-----|-----|-----
| [CVE-2002-0013](https://www.cvedetails.com/cve/CVE-2002-0013/) | Admin | Low 
| [CVE-2002-0012](https://www.cvedetails.com/cve/CVE-2002-0012/) | Admin | Low 
| [CVE-1999-0472](https://www.cvedetails.com/cve/CVE-1999-0472/) | None |  Low

### List of CVE Security Vulnerabilities for SNMP Version 3

|CVE ID|
|----|
|[CVE-2020-3235](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-3235)|
|[CVE-2019-20892](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-20892)|
|[CVE-2018-15328](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-15328)|
|[CVE-2018-0161](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-0161)|
|[CVE-2017-7737](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-7737)|
|[CVE-2016-8562](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-8562)|
|[CVE-2014-9695](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-9695)|
|[CVE-2013-4631](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2013-4631)|
|[CVE-2013-4630](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2013-4630)|
|[CVE-2013-3634](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2013-3634)|
|[CVE-2012-4838](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2012-4838)|
|[CVE-2009-0625](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2009-0625)|
|[CVE-2008-7095](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2008-7095)|
|[CVE-2008-4594](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2008-4594)|
|[CVE-2008-0960](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2008-0960)|
|[CVE-2003-1003](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2003-1003)|
|[CVE-2003-1002](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2003-1002)|

### References and Articles

- [Multiple Vulnerabilities in SNMP](https://www.computer.org/csdl/magazine/co/2002/04/r4s02/13rRUIIVl7v)
- [Security issues and vulnerabilities of the SNMP protocol](https://www.ee.ucl.ac.uk/lcs/previous/LCS2003/102.pdf)
- [SNMP Version 3 Authentication Vulnerabilities](https://tools.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-20080610-snmpv3)


