# Cloud Analytics Lab

## Lab Topology

A Cloud Analytics lab provides an environment for Grafana to use Prometheus as a data source for time-series data and for Prometheus to scrape metrics from exporters. The basic OSS components of the architecture are arranged in simple lab topology:

<img src="_media/lab-topology.png" alt="Lab Topology" style="display: block; margin-left: auto; margin-right: auto; height: 80%; width: 80%; padding-top: 50px; padding-bottom: 25px;" />

## A. Lab Devices

|Device|IP| OS | 
|---------|-------|-------|
|Machine 1|206.253.238.53| RHEL
|Machine 2|206.253.238.54| RHEL
|Machine 3|10.142.189.180| Windows (NAT IP: 206.253.238.184)
|Machine 4|206.253.228.150| RHEL - Guardium Console 

## B. Lab Setup

The lab environment runs 1) Grafana, 2) Grafana Image Renderer, and 3) Prometheus as Docker containers. A prerequisite for creating the lab is to install the [Docker Engine](https://docs.docker.com/engine/). The following sections provide details for each application startup. The other components that run in the lab are [Prometheus Exporters](https://prometheus.io/docs/instrumenting/exporters/), which run containerized or as stand-alone processes. The hosts in the lab may  also connect to and communicate with devices outside of the lab as needed for integration and testing purposes. 

### 1. Grafana

#### 1.1 Docker Volumes

Without a designated location for container data persistence, all Grafana configuration and dashboard data is held by the container and doesn’t persist when a container is destroyed. To save container state, a persistent storage or volume should be defined per instance of Grafana:

```
sudo docker volume create grafana-storage1
```
```
sudo docker volume create grafana-storage2
```
```
sudo docker volume create grafana-storage3
```

#### 1.2 HTTPS

To enable HTTPS, steps to generate a self-signed certificate for Grafana to use:

http://www.turbogeek.co.uk/2020/09/30/grafana-how-to-configure-ssl-https-in-grafana/

Create an SSL Certificate

```
> cd cloud-analytics-poc/grafana
```

```
openssl genrsa -out grafana.key 2048
```

Generate a certificate signing request

```
openssl req -new -key grafana.key -out grafana.csr
```

Output the certificate

```
openssl x509 -req -days 365 -in grafana.csr -signkey grafana.key -out grafana.crt
```

#### 1.3 Enable SSL for Grafana

The following parameters and environment variables enable SSL for Grafana. 

**Grafana 1**

```
cd cloud-analytics-poc
```

```
sudo docker run --user root -d --restart unless-stopped -p 3000:3000 \
--name grafana1 -v grafana-storage1:/var/lib/grafana \ 
-e "GF_SERVER_HTTP_PORT=3000" -e "GF_SECURITY_ALLOW_EMBEDDING=true" \ 
-e "GF_SECURITY_COOKIE_SAMESITE=disabled" -e "GF_AUTH_ANONYMOUS_ENABLED=true" \ 
-e "GF_SERVER_PROTOCOL=https" -e "GF_SERVER_CERT_FILE=/var/lib/grafana/grafana.crt" \ 
-e "GF_SERVER_CERT_KEY=/var/lib/grafana/grafana.key" \ 
-v "$(pwd)"/grafana/grafana.crt:/var/lib/grafana/grafana.crt \ 
-v "$(pwd)"/grafana/grafana.key:/var/lib/grafana/grafana.key grafana/grafana
```

**Grafana 2**

```
sudo docker run --user root -d --restart unless-stopped -p 3001:3001 \
--name grafana2 -v grafana-storage2:/var/lib/grafana \ 
-e "GF_SERVER_HTTP_PORT=3001" -e "GF_SECURITY_ALLOW_EMBEDDING=true" \ 
-e "GF_SECURITY_COOKIE_SAMESITE=disabled" -e "GF_AUTH_ANONYMOUS_ENABLED=true" \ 
-e "GF_SERVER_PROTOCOL=https" -e "GF_SERVER_CERT_FILE=/var/lib/grafana/grafana.crt" \ 
-e "GF_SERVER_CERT_KEY=/var/lib/grafana/grafana.key" \ 
-v "$(pwd)"/grafana/grafana.crt:/var/lib/grafana/grafana.crt \ 
-v "$(pwd)"/grafana/grafana.key:/var/lib/grafana/grafana.key grafana/grafana
```

**Grafana 3**

```
sudo docker run --user root -d --restart unless-stopped -p 3002:3002 \
--name grafana3 -v grafana-storage3:/var/lib/grafana \ 
-e "GF_SERVER_HTTP_PORT=3002" -e "GF_SECURITY_ALLOW_EMBEDDING=true" \ 
-e "GF_SECURITY_COOKIE_SAMESITE=disabled" -e "GF_AUTH_ANONYMOUS_ENABLED=true" \ 
-e "GF_SERVER_PROTOCOL=https" -e "GF_SERVER_CERT_FILE=/var/lib/grafana/grafana.crt" \ 
-e "GF_SERVER_CERT_KEY=/var/lib/grafana/grafana.key" \ 
-v "$(pwd)"/grafana/grafana.crt:/var/lib/grafana/grafana.crt \ 
-v "$(pwd)"/grafana/grafana.key:/var/lib/grafana/grafana.key grafana/grafana
```

#### 1.4 Rendering Server

Grafana will automatically render panels and dashboards as PNG images. This is done through an image renderer that uses headless Chromium. To setup a Grafana instance to communicate with an image renderer, the following parameters and environment variables are used:

**Grafana 1**

```
> cd cloud-analytics-poc
```

```
sudo docker run --user root -d --restart unless-stopped -p 3000:3000 --name grafana1 -v grafana-storage1:/var/lib/grafana \
-e "GF_SERVER_HTTP_PORT=3000" -e "GF_SECURITY_ALLOW_EMBEDDING=true" \
-e "GF_SECURITY_COOKIE_SAMESITE=disabled" -e "GF_AUTH_ANONYMOUS_ENABLED=true" \
-e "GF_SERVER_PROTOCOL=https" -e "GF_SERVER_CERT_FILE=/var/lib/grafana/grafana.crt" \
-e "GF_SERVER_CERT_KEY=/var/lib/grafana/grafana.key" \
-e "GF_RENDERING_SERVER_URL=http://dal09-dev-analytics-01a.sec.ibm.com:8081/render" \
-e "GF_RENDERING_CALLBACK_URL=https://dal09-dev-analytics-01a.sec.ibm.com:3000/" \
-e "GF_LOG_FILTERS=rendering:debug" \
-v "$(pwd)"/grafana/grafana.crt:/var/lib/grafana/grafana.crt \
-v "$(pwd)"/grafana/grafana.key:/var/lib/grafana/grafana.key grafana/grafana
```

#### 1.5 Grafana Renderer

The following runs an image renderer as a remote image rendering service in a separate Docker container. 

```
sudo docker run --user root -d --restart unless-stopped -p 8081:8081 --name renderer  -e "ENABLE_METRICS=true" -e "IGNORE_HTTPS_ERRORS=true" grafana/grafana-image-renderer
```

### 2. Prometheus

Two Docker volumes are created for persistent storage of data collected by Prometheus:

```
sudo docker volume create prom-volume1
```
```
sudo docker volume create prom-volume2
```
```
sudo docker volume create prom-volume3
```

To start Prometheus containers:

**Prometheus 1**

```
sudo docker run --user root -d --restart unless-stopped -p 9090:9090  --name prometheus1 -v prom-volume1:/prometheus-data \
-v "$(pwd)"/prometheus1/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus \
--web.listen-address=:9090 --config.file=/etc/prometheus/prometheus.yml
```

**Prometheus 2**

```
sudo docker run --user root -d --restart unless-stopped -p 9191:9191 --name prometheus2 \
-v prom-volume2:/prometheus-data \
-v "$(pwd)"/prometheus2/prometheus.yml:/etc/prometheus/prometheus.yml \
-v "$(pwd)"/prometheus2/d09-dev-iam-01b-cert.pem:/etc/prometheus/d09-dev-iam-01b-cert.pem \ 
-v "$(pwd)"/prometheus2/d09-dev-iam-01b-key.pem:/etc/prometheus/d09-dev-iam-01b-key.pem prom/prometheus \
--web.listen-address=:9191 --config.file=/etc/prometheus/prometheus.yml
```

**Prometheus 3**

```
sudo docker run --user root -d --restart unless-stopped -p 9292:9292 --name prometheus3 -v prom-volume3:/prometheus-data \
-v "$(pwd)"/prometheus3/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus \ 
--web.listen-address=:9292 --config.file=/etc/prometheus/prometheus.yml
```

### 2.1 Prometheus Config

Prometheus is configured by a configuration file. The configuration file defines scraping jobs and their instances. A configuration reload is triggered by sending an HTTP POST request to the /-/reload endpoint (when the --web.enable-lifecycle flag is enabled). 

**Prometheus 1**

The "$(pwd)"/prometheus2/prometheus.yml file for prometheus1: 

```
# global config
global:
  scrape_interval:     60s # Set the scrape interval to every 60 seconds. Default is every 1 minute.
  evaluation_interval: 60s # Evaluate rules every 60 seconds. The default is every 1 minute.
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
    - targets: ['localhost:9090']
  - job_name: 'node_exporter'
    static_configs:
    - targets: ['206.253.238.53:9100']
```

**Prometheus 2**

The "$(pwd)"/prometheus2/prometheus.yml file for prometheus2: 

```
# global config
global:
  scrape_interval:     60s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 60s # Evaluate rules every 15 seconds. The default is every 1 minute.
scrape_configs:
  - job_name: 'windows_node_exporter'
    scheme: https
    tls_config:
      cert_file: /etc/prometheus/d09-dev-iam-01b-cert.pem
      key_file: /etc/prometheus/d09-dev-iam-01b-key.pem
      insecure_skip_verify: true 
    static_configs:
    - targets: ['206.253.238.184']
```

**Prometheus 3**

The "$(pwd)"/prometheus3/prometheus.yml file for prometheus3: 

```
scrape_configs:
  - job_name: 'snmp'
    static_configs:
      - targets:
        - 206.253.238.53  # SNMP device.
    metrics_path: /snmp
    params:
      module: [default]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 206.253.238.53:9116  # The SNMP exporter's real hostname:port.
```

### 3. Prometheus Node Exporter

Node Exporter is a Prometheus exporter for hardware and OS metrics with pluggable metric collectors. An exporter allows for the measurement of various machine resources such as memory, disk, and CPU utilization. 

To install node_exporter on the primary lab device, 206.253.238.53:

```
wget https://github.com/prometheus/node_exporter/releases/download/v*/node_exporter-*.*-amd64.tar.gz
tar xvfz node_exporter-*.*-amd64.tar.gz
cd node_exporter-*.*-amd64
```

To start node_exporter on the primary lab device and separate the process from the terminal:

```
nohup ./node_exporter &>> node_exporter1.log&
disown
```

To start node_exporter on the secondary lab device, 206.253.238.54:

```
nohup ./node_exporter &>> node_exporter2.log&
disown
```

Any additional exporter may become another "target" for Prometheus, by adding an 1.2.3.4:5555 to the `scrape_configs` section in prometheus.yml for the `node_exporter` job, where 1.2.3.4 is the IP of the machine hosting the exporter process on the device to get metrics from, and 5555 is the port it listens on for scrape requests. For example:

```
scrape_configs:
...
  - job_name: 'node_exporter'
    static_configs:
    - targets: ['206.253.238.54:9100', '10.142.189.180:9182', `1.2.3.4:5555`]
```

### 4. Setup SNMP on RHEL

A basic intallation and configuration procedure for SNMP on a Linux host. The steps below setup SNMP on the 206.253.238.53 host in the lab to enable self-monitoring. 

Install SNMP

```
yum -y install net-snmp net-snmp-utils
```

Backup snmp.conf

```
cp /etc/snmp/snmpd.conf /etc/snmp/snmpd.conf.orig
```

Clear out the contents of the file:

```
rm /etc/snmp/snmpd.conf
touch /etc/snmp/snmpd.conf
```

Add to /etc/snmp/snmpd.conf a single line to allow an IP to monitor this one:

```
rocommunity random_string monitoring_server_ip
```

Where `random_string` is any random community string and `monitoring_server_ip` is the host allowed to send snmp requests. 

For example, to allow self-monitoring:

```
rocommunity public 206.253.238.53
```

After any updates to snmpd.conf, restart the snmp daemon:

```
/bin/systemctl restart snmpd
```

After a restart, snmpwalk should work:

```
snmpwalk -c public -v2c 206.253.238.53
SNMPv2-MIB::sysDescr.0 = STRING: Linux dal09-dev-analytics-01a 3.10.0-1160.15.2.el7.x86_64 #1 SMP Thu Jan 21 16:15:07 EST 2021 x86_64
SNMPv2-MIB::sysObjectID.0 = OID: NET-SNMP-MIB::netSnmpAgentOIDs.10
DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (1462) 0:00:14.62
SNMPv2-MIB::sysContact.0 = STRING: root@localhost
SNMPv2-MIB::sysName.0 = STRING: dal09-dev-analytics-01a
SNMPv2-MIB::sysLocation.0 = STRING: Unknown
SNMPv2-MIB::sysORLastChange.0 = Timeticks: (3) 0:00:00.03
SNMPv2-MIB::sysORID.1 = OID: SNMP-MPD-MIB::snmpMPDCompliance
```

```
export MIBDIRS=/usr/share/snmp/mibs
snmpwalk -c public -v2c 206.253.238.53 HOST-RESOURCES-MIB::hrStorage
```

### 5. Prometheus SNMP Exporter Setup

The [SNMP Exporter](https://github.com/prometheus/snmp_exporter) exposes information gathered from SNMP for use by the Prometheus. The following steps setup the SNMP Exporter to monitor a Linux host. 

Build the generator

```
# Debian-based distributions.
> sudo apt-get install unzip build-essential libsnmp-dev p7zip-full # Debian-based distros
# Redhat-based distributions.
> sudo yum install gcc gcc-g++ make net-snmp net-snmp-utils net-snmp-libs net-snmp-devel # RHEL-based distros

> go get github.com/prometheus/snmp_exporter/generator
> cd ${GOPATH-$HOME/go}/src/github.com/prometheus/snmp_exporter/generator
> go build
> make mibs
```

Generate an snmp.yml file from generator.yml

```
> cat generator.yml
modules:
  default:
    walk:
      - ssCpuIdle
      - memTotalFree
      - memTotalReal
      - memAvailReal
      - hrStorageUsed
      - hrStorageAllocationUnits
      - hrStorageDescr
      - hrStorageSize
      - hrProcessorTable
      - sysUpTime
      - sysName
    auth:
      community: public
```

Run the generator to generate an `snmp.yml` file:

```
export MIBDIRS=/usr/share/snmp/mibs
./generator generate
```

Download the SNMP Exporter binary and install:

https://github.com/prometheus/snmp_exporter/releases

Copy `snmp.yml` from the generator directory to the SNMP Exporter installation directory:

```
cp snmp.yml "$(pwd)"/snmp_exporter-0.19.0.linux-amd64
```

Start the SNMP Exporter process from its installation directory:

```
> nohup ./snmp_exporter &> snmp_exporter.log&
> disown
> cat snmp_exporter.log
level=info ts=2021-02-16T08:13:32.047Z caller=main.go:149 msg="Starting snmp_exporter" version="(version=0.19.0, branch=HEAD, revision=9dcbc02f59648b21fcf632de1b62a30df70f4649)"
level=info ts=2021-02-16T08:13:32.047Z caller=main.go:150 build_context="(go=go1.14.7, user=root@387afaad41d6, date=20200831-12:07:03)"
level=info ts=2021-02-16T08:13:32.062Z caller=main.go:243 msg="Listening on address" address=:9116
```

Go to the SNMP Exporter's `/snmp` endpoint, specify a `target` to query and module `default` :

http://206.253.238.53:9116/snmp?target=206.253.238.53&module=default

The output from this endpoint to be scraped by Prometheus. 

### 5.1 SNMP Exporter Docker Setup

Start the Docker container

```
sudo docker run --user root -d --restart unless-stopped -p 9117:9116 --name snmp_exporter prom/snmp-exporter 
```

Verify startup

```
sudo docker logs snmp_exporter

level=info ts=2021-03-10T03:35:56.312Z caller=main.go:152 msg="Starting snmp_exporter" version="(version=0.20.0, branch=HEAD, revision=c33572b6c8c8e43a479fde0f9fa8ac86e15598bc)"
level=info ts=2021-03-10T03:35:56.312Z caller=main.go:153 build_context="(go=go1.15.8, user=root@eebd39e6960e, date=20210212-11:37:48)"
level=info ts=2021-03-10T03:35:56.530Z caller=main.go:246 msg="Listening on address" address=:9116
level=info ts=2021-03-10T03:35:56.535Z caller=tls_config.go:191 msg="TLS is disabled." http2=false
```

Verify output from endpoint

http://206.253.238.53:9117/

View Default Config (snmp.yml)

http://206.253.238.53:9117/config

Shell into container and grep the snmp.yml file:

```
sudo docker exec -it snmp_exporter cat /etc/snmp_exporter/snmp.yml
```

### 6. Windows Exporter Setup

https://github.com/prometheus-community/windows_exporter

Windows Host for Grafana / Prometheus 2

http://206.253.238.184:9182/metrics

**Note:** Per Infra Team, Windows Host `10.142.189.180` is located behind a load balancer and nated toward rest of the network on address `10.253.238.184`. So logicaly they are both in the same network, not separated by a firewall. To reach `10.142.189.180` from the primary lab device, `206.253.238.53`, you need to use its NAT IP: `206.253.238.184`. 

#### Create a Windows Service 

```
C:\Users\arodriguez\Downloads\windows_exporter>sc.exe create "Windows Exporter" binPath="C:\Users\arodriguez\Downloads\windows_exporter\windows_exporter.exe" start=auto

[SC] CreateService SUCCESS

```

Windows Administrative Tools > Services > Windows Exporter > Properties

General > Startup Type: Automatic 

Recovery

First failure: Restart the Service   
Second failure: Restart the Service  
Subsequent failures: Restart the Service  

### 7. Grafana Dashboards & Prometheus Data Sources

Import the "Node Exporter Full" dashboard into Grafana

https://grafana.com/grafana/dashboards/1860

And create Prometheus Data Sources for each of:

**Grafana 1**

Prometheus 1: http://206.253.238.53:9090/

**Grafana 2**

Prometheus 2: http://206.253.238.53:9191/

**Grafana 3**

Prometheus 3: http://206.253.238.53:9292/

## C. Lab Links

### Grafana 1:

https://206.253.238.53:3000/

User: admin

#### Node Exporter Full Dashboard

https://206.253.238.53:3000/d/rYdddlPWk/node-exporter-full?orgId=1&refresh=1m&from=now-5m&to=now

### Grafana 2:

https://206.253.238.53:3001/

User: admin

#### Windows Exporter Metrics

https://206.253.238.53:3001/d/AqYF52yGk/windows-exporter-metrics?orgId=1&refresh=1m&from=now-5m&to=now


### Grafana 3:

https://206.253.238.53:3002/

User: admin

#### SNMP Exporter Dashboard

https://206.253.238.53:3000/d/kEPzV9EGz/snmp-metrics

### Prometheus 1:

http://206.253.238.53:9090/

http://206.253.238.53:9090/targets

http://206.253.238.53:9090/metrics

### Prometheus 2:

http://206.253.238.53:9191/

http://206.253.238.53:9191/targets

http://206.253.238.53:9191/metrics

### Prometheus 3 (SNMP):

http://206.253.238.53:9292/

http://206.253.238.53:9292/targets

http://206.253.238.53:9292/metrics

## D. Perf Tests

The following cmds will place load on the CPU and allocate memory arbitarilly for testing purposes and to verify metrics for CPU utilization and memory are accurately reflected in dashboards after data acquisition is setup for a host. 

Use 100% of 1 CPU Core 

```
cat /dev/zero > /dev/null
```

Allocate 20GB memory

```
</dev/zero head -c 20000m | tail
```

### Memory

The `free -m` cmd will return all details related to memory allocation. 

```
> free -mh
              total        used        free      shared  buff/cache   available
Mem:            31G        1.5G         28G        408K        1.6G         29G
Swap:          4.0G        229M        3.8G
```

### Disk Space

The `df` cmd will return all details related to disk allocation. 

```
> df -H --o
Filesystem                            Type     Inodes IUsed IFree IUse%  Size  Used Avail Use% File Mounted on
devtmpfs                              devtmpfs   4.1M   383  4.1M    1%   17G     0   17G   0% -    /dev
tmpfs                                 tmpfs      4.1M     1  4.1M    1%   17G     0   17G   0% -    /dev/shm
tmpfs                                 tmpfs      4.1M  1.6k  4.1M    1%   17G  4.7M   17G   1% -    /run
tmpfs                                 tmpfs      4.1M    16  4.1M    1%   17G     0   17G   0% -    /sys/fs/cgroup
/dev/mapper/vg0-root                  ext4       961k   76k  886k    8%   16G  2.3G   13G  16% -    /
/dev/sda1                             ext4        66k   345   66k    1%  1.1G  189M  764M  20% -    /boot
/dev/mapper/vg0-var                   ext4       641k   54k  588k    9%   11G  7.2G  2.6G  74% -    /var
/dev/mapper/vg0-opt                   ext4       4.3M   66k  4.3M    2%   70G  1.1G   65G   2% -    /opt
206.253.233.15:/export/mss_apps       nfs        246M   13M  234M    6%  8.0T  6.5T  1.2T  85% -    /mss_apps
206.253.233.15:/export/app_logs       nfs        246M   13M  234M    6%  8.0T  6.5T  1.2T  85% -    /app_logs
206.253.233.15:/export/dal09-dev-home nfs        246M   13M  234M    6%  8.0T  6.5T  1.2T  85% -    /home
tmpfs                                 tmpfs      4.1M     1  4.1M    1%  3.4G     0  3.4G   0% -    /run/user/1331
```

### Task Manager

The `top` cmd will display information about CPU and memory utilization plus a detailed list of processes and resource utilization per process. 

```
> top
top - 00:29:35 up 10 days, 16:51,  6 users,  load average: 1.00, 1.01, 1.05
Tasks: 221 total,   1 running, 220 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni, 99.9 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 32761636 total, 29463820 free,  1618844 used,  1678972 buff/cache
KiB Swap:  4194300 total,  3959616 free,   234684 used. 30739632 avail Mem 

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                     
 86461 arodrig+  20   0  715012  20400   3972 S   0.7  0.1   0:01.58 snmp_exporter                                                                                               
   723 root      20   0       0      0      0 S   0.3  0.0   0:24.45 jbd2/dm-3-8                                                                                                 
 18592 root      20   0  760516  22224   7164 S   0.3  0.1   1:27.21 grafana-server                                                                                              
     1 root      20   0  191452   3256   1656 S   0.0  0.0   4:09.76 systemd                                                                                                     
     2 root      20   0       0      0      0 S   0.0  0.0   0:00.09 kthreadd                                                                                                    
     4 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 kworker/0:0H                                                                                                
     6 root      20   0       0      0      0 S   0.0  0.0   0:02.00 ksoftirqd/0            
```

<hr/>

<footer>
<span>Page Last Updated: {docsify-updated}</span>
</footer>
