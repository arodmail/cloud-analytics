# Operational Management

> For more information see the [Cloud Management](https://en.wikipedia.org/wiki/Cloud_management).


## On-Boarding Flow

<img src="_media/onboarding.png" alt="On-Boarding" style="display: block; margin-left: auto; margin-right: auto; height: 70%; width: 70%; padding-top: 25px; padding-bottom: 25px;"/>

1. Install exporter on endpoint, configure security (TLS), start exporter, listens on secure port
2. Deploy Prometheus customer time-series database instance into customer namespace, mount persistent customer volume 
3. Deploy Grafana customer instance into customer namespace, mount peristent customer volume, define Prometheus as a Grafana data source. 
4. Add Misc Info name/value pairs to `OPS:Device Details`record to include device specific parameters to pass into applications that embed Grafana urls for dashboards and/or panels. 

## Off-Boarding Flow

<img src="_media/offboarding.png" alt="Off-Boarding" style="display: block; margin-left: auto; margin-right: auto; height: 70%; width: 70%; padding-top: 25px; padding-bottom: 25px;"/>

1. Deactive Device in Remedy
2. Shut down customer Grafana container
3. Shut down customer Prometheus container
4. Uninstall exporter from endpoint
5. Remove customer data from persistent storage on the NAS/NFS

## On-Boarding Manual Procedure

### Step 1

Login to OpenShift

| Cluster | Web console URL |
| --------  | ----------------- |
| Dev | https://console-openshift-console.apps-priv.dal09-dev-ocp-01.cl.sec.ibm.com/ 
| Stg | https://console-openshift-console.apps-priv.atl-stg-ocp-01.cl.sec.ibm.com/
| Prod | https://console-openshift-console.apps-priv.atl-prd-ocp-01.cl.sec.ibm.com/

### Step 2 

Create a Project/Namespace for Customer

Projects > Create Project. The following naming convention is recommended:

| Customer Name | Customer ID |  Project Name | Display Name | Description 
| --------  | -------- | -------- |  -------- | -------- |
| ACME Inc. | CID001287 | **cid001287-cloud-analytics** | cid001287-cloud-analytics | A Cloud Analytics namespace for Customer ID: CID001287

Where `cid001287` is the Customer ID all lower-case, and `-cloud-analytics` is a fixed string. 

**Note:** An OpenShift "Project Name" DNS-1123 label must consist of lower case alphanumeric characters or '-', and must start and end with an alphanumeric character (e.g. 'my-name',  or '123-abc', regex used for validation is '[a-z0-9]([-a-z0-9]*[a-z0-9])?')" for field "metadata.name".

### Step 3

Deploy Grafana

- a. Switch from "Administrator" to "Developer" Perspective 
- b. Project > **cid001287-cloud-analytics**
- c. +Add > Container Image 
- d. Image name from external registry > `grafana/grafana` (Wait for "Validated" messsage)
- e. General > Application Name: **grafana-app** / Name: **grafana**
- f. Resources (Select the resource type to generate): Deployment 
- g. Advanced Options > Create a route to the application = true
- h. Click on the names to access advanced options for "Routing"
- i. Hostname: (blank) | Path: / | Target Port: 3000 | Security: Secure Route = true | TLS Termination: Edge | Insecure Traffic: Allow
- j. Create

**Note on TLS Termination:** With edge termination, TLS termination occurs at the router, prior to proxying traffic to its destination. TLS certificates are served by the front end of the router, so they must be configured into the route, otherwise the router’s default certificate will be used for TLS termination.

### Step 4

Verify Grafana Route accepts requests 

- a. Switch from "Developer" to "Administrator" Perspective 
- b. Networking > Routes 
- c. Project > **cid001287-cloud-analytics**
- d. Name: grafana-route > Location > https://grafana-cid001287-cloud-analytics.apps-priv.atl-stg-ocp-01.cl.sec.ibm.com/
- e. Verify Grafana Login Page is 200 Ok. 
- f. Login to Grafana: Username > admin / Password > admin ("Skip" change password)

### Step 5

Add Storage for Grafana

- a. Switch from "Developer" to "Administrator" Perspective
- b. Workloads > Deployments
- c. Project > **cid001287-cloud-analytics**
- d. Grafana > Actions > Add Storage
- e. Create new Claim > Storage Class "(default) ocs-storagecluster-cephfs"
- f. Persistent Volume Claim Name: `grafana-storage-claim`
- g. Access Mode: Single User (RWO)
- h. Size: 1 GiB
- i. Mount Path: `/grafana-data`
- j. Save

### Step 6 

Deploy Prometheus

- a. Switch from "Administrator" to "Developer" Perspective 
- b. Project > **cid001287-cloud-analytics**
- c. +Add > Container Image 
- d. Image name from external registry > `prom/prometheus` (Wait for "Validated" messsage)
- e. General > Application Name: **prometheus-app** / Name: **prometheus**
- f. Resources (Select the resource type to generate): Deployment 
- g. Advanced Options > Create a route to the application = true
- h. Create

### Step 7 

Verify Prometheus is up and running in the customer namespace

- a. Switch from "Developer" to "Administrator" Perspective 
- b. Networking > Routes
- c. Project > **cid001287-cloud-analytics**
- d. Name: **prometheus** > Location > http://prometheus-cid001287-cloud-analytics.apps-priv.atl-stg-ocp-01.cl.sec.ibm.com/
- e. Verify Prometheus Graph UI is 200 Ok. 

### Step 8

Add Storage for Prometheus

- a. Switch from "Developer" to "Administrator" Perspective
- b. Workloads > Deployments
- c. Project > **cid001287-cloud-analytics**
- d. Prometheus > Actions > Add Storage
- e. Create new Claim > Storage Class "(default) ocs-storagecluster-cephfs" 
- f. Persistent Volume Claim Name: `prom-storage-claim`
- g. Access Mode: Single User (RWO)
- h. Size: 1 GiB
- i. Mount Path: `/prometheus-data`
- j. Save

### Step 9

Add Prometheus Configuration (for Windows Node Exporter as a target)

- a. Switch from "Developer" to "Administrator" Perspective
- b. Workloads > Pods
- c. Project > **cid001287-cloud-analytics**
- d. prometheus > Terminal 
- e. `cd /prometheus-data`
- f. Create the file `prometheus.yml` (See Note 1)

```
vi prometheus.yml

# global config
global:
  scrape_interval:     60s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 60s # Evaluate rules every 15 seconds. The default is every 1 minute.
scrape_configs:
  - job_name: 'windows_node_exporter'
    scheme: https
    tls_config:
      cert_file: /prometheus-data/{windows-host-name}-cert.pem
      key_file: /prometheus-data/{windows-host-name}-key.pem
      insecure_skip_verify: true 
    static_configs:
    - targets: ['{windows-host-ip}:8443']

```
- g. Save the file `:wq`
- h. Repeat steps (f) and (g) for X.509 certificate/key pair `{windows-host-name}-cert.pem`, and `{windows-host-name}-key.pem` (See Note 2)
- i. Workloads > Deployments > prometheus > Details
- j. Recycle the Pod: Decrease pod count to 0 > Increase pod count to 1
- k. Confirm success

**Note 1**: A Prometheus configuration statically targets an endpoint to scrape. In the example above to scrape statistics from a Windows host, `{windows-host-ip}:8443` should be replaced with the IP address of a host running Windows Node Exporter, listening on port 8443 for HTTPS scrape requests. See [Install/Deploy Windows Node Exporter](https://pages.github.ibm.com/security-services-platform/squad_phoenix/#/windows_exporter) for installation/configuration details. 

**Note 2**: The X.509 certificate/key pair files may be exported from the monitored host.  

### Step 10

Update Prometheus Deployment 

- a. Switch from "Developer" to "Administrator" Perspective
- b. Workloads > Deployments
- c. Project > **cid001287-cloud-analytics**
- d. prometheus
- e. Decrease the pod count to 0 
- f. YAML 
- g. Go to section: `spec` > `template` > `spec` > `containers`
- h. Add the following `args`:

```
          args:
            - '--config.file=/prometheus-data/prometheus.yml' 
            - '--storage.tsdb.path=/prometheus-data/'
```

- i. Save
- j. Details > Increase the pod count to 1
- k. Confirm success

### Step 11

Create a Prometheus Data Source in Grafana

- a. Repeat Step 4
- b. Configuration > Data Sources > Add Data Source > Prometheus > Select
- c. HTTP URL: From Step 7d
- d. Save & Test > "Data source is working" 

### Step 12

Install the Windows Node Exporter Full Dashboard into Grafana

- a. Repeat Step 4
- b. Dashboards Manage > Import > Dashboard ID: `10171` > Load
- c. Options > Name: Node Exporter Full
- d. Prometheus > Select a Data Source > Prometheus (default)
- e. Import
- f. Verify Full Dashboard is displayed 200 Ok. 

### Step 13

Deploy Grafana Image Renderer

- a. Switch from "Administrator" to "Developer" Perspective 
- b. Project > **cid001287-cloud-analytics**
- c. +Add > Container Image 
- d. Image name from external registry > `grafana/grafana-image-renderer` (Wait for "Validated" messsage)
- e. General > Application Name: **grafana-image-renderer-app** / Name: **grafana-image-renderer**
- f. Resources (Select the resource type to generate): Deployment 
- g. Advanced Options > Create a route to the application = true
- h. Click on the names to access advanced options for "Routing"
- i. Hostname: (blank) | Path: / | Target Port: 8081 | Security: Secure Route = false 
- j. Create

### Step 14

Copy the Route URL for Grafana Image Renderer

- a. Switch from "Developer" to "Administrator" Perspective 
- b. Networking > Routes
- c. Project > **cid001287-cloud-analytics**
- d. Name: **grafana-image-renderer** > Location > http://grafana-image-renderer-cid001287-cloud-analytics.apps-priv.atl-stg-ocp-01.cl.sec.ibm.com/

#### Sample OpenShift Routes

<img src="_media/open-shift-routes.png" alt="OpenShift Routes" style="display: block; margin-left: auto; margin-right: auto; height: 80%; width: 80%; padding-top: 25px; padding-bottom: 25px;"/>

### Step 15

Edit Grafana Deployment to define environment variables

- a. Switch from "Developer" to "Administrator" Perspective
- b. Workloads > Deployments
- c. Project > **cid001287-cloud-analytics**
- d. grafana > Environment 

Single values (env)

| Name | Value | Notes
| --------  | ---------- | ---------- |
| GF_SECURITY_ALLOW_EMBEDDING | true  | Allows panel embedding.  
| GF_SECURITY_COOKIE_SAMESITE | disabled | Disables CSRF. More details [here](https://grafana.com/docs/grafana/latest/administration/configuration/#cookie_samesite). 
| GF_AUTH_ANONYMOUS_ENABLED | true | Allows unauthenticated requests from microservices. Note that as a namespace-scoped service without an Ingress, Grafana is only exposed to other microservices in the cluster and not externally.
| GF_RENDERING_SERVER_URL | http://grafana-image-renderer-cid001287-cloud-analytics.apps-priv.atl-stg-ocp-01.cl.sec.ibm.com/render | _Copy from Step 14d_ 
| GF_RENDERING_CALLBACK_URL | https://grafana-cid001287-cloud-analytics.apps-priv.atl-stg-ocp-01.cl.sec.ibm.com/ |  _Copy from Step 4d_
| GF_LOG_FILTERS | rendering:debug | Enables debug logging for the Grafana renderer
| GF_PATHS_DATA | `/grafana-data` | Defines a data directory for Grafana persistence, e.g., dashboard saves. 

- e. Save

### Step 16

Edit Grafana Image Renderer Deployment to define environment variables

- a. Switch from "Developer" to "Administrator" Perspective
- b. Workloads > Deployments
- c. Project > **cid001287-cloud-analytics**
- d. grafana-image-renderer > Environment 

Single values (env)

| Name | Value | Notes
| --------  | ---------- | ---------- |
| ENABLE_METRICS | true  | Exposes a Prometheus /metrics endpoint.
| IGNORE_HTTPS_ERRORS | true | Skips https errors, if any. 

- e. Save

#### Sample Deployment Environment for Grafana

<img src="_media/grafana-deployment-environment.png" alt="Sample Grafana Deployment" style="display: block; margin-left: auto; margin-right: auto; height: 80%; width: 80%; padding-top: 25px; padding-bottom: 25px;"/>

### Step 18 

Deploy SNMP Exporter

- a. Switch from "Administrator" to "Developer" Perspective 
- b. Project > **cid001287-cloud-analytics**
- c. +Add > Container Image 
- d. Image name from external registry > `prom/snmp-exporter` (Wait for "Validated" messsage)
- e. General > Application Name: **snmp-exporter-app** / Name: **snmp-exporter**
- f. Resources (Select the resource type to generate): Deployment 
- g. Advanced Options > Create a route to the application = true
- h. Create

### Step 19

Add Storage for SNMP Exporter

- a. Switch from "Developer" to "Administrator" Perspective
- b. Workloads > Deployments
- c. Project > **cid001287-cloud-analytics**
- d. snmp-exporter > Actions > Add Storage
- e. Create new Claim > Storage Class "(default) ocs-storagecluster-cephfs" 
- f. Persistent Volume Claim Name: `snmp-exporter-storage-claim`
- g. Access Mode: Single User (RWO)
- h. Size: 1 GiB
- i. Mount Path: `/snmp-data`
- j. Save

### Step 20

Add SNMP Configuration 

- a. Switch from "Developer" to "Administrator" Perspective
- b. Workloads > Pods
- c. Project > **cid001287-cloud-analytics**
- d. snmp-exporter > Terminal 
- e. `cd /snmp-data`
- f. Create the file `snmp.yml`

```
vi snmp.yml

default:
  walk:
  - 1.3.6.1.2.1.25.2.3.1.3
  - 1.3.6.1.2.1.25.2.3.1.4
  - 1.3.6.1.2.1.25.2.3.1.5
  - 1.3.6.1.2.1.25.2.3.1.6
  - 1.3.6.1.2.1.25.3.3
  get:
  - 1.3.6.1.2.1.1.3.0
  - 1.3.6.1.2.1.1.5.0
  - 1.3.6.1.4.1.2021.11.11.0
  - 1.3.6.1.4.1.2021.4.11.0
  - 1.3.6.1.4.1.2021.4.5.0
  - 1.3.6.1.4.1.2021.4.6.0
  metrics:
  - name: sysUpTime
    oid: 1.3.6.1.2.1.1.3
    type: gauge
    help: The time (in hundredths of a second) since the network management portion
      of the system was last re-initialized. - 1.3.6.1.2.1.1.3
  - name: sysName
    oid: 1.3.6.1.2.1.1.5
    type: DisplayString
    help: An administratively-assigned name for this managed node - 1.3.6.1.2.1.1.5
  - name: hrStorageDescr
    oid: 1.3.6.1.2.1.25.2.3.1.3
    type: DisplayString
    help: A description of the type and instance of the storage described by this
      entry. - 1.3.6.1.2.1.25.2.3.1.3
    indexes:
    - labelname: hrStorageIndex
      type: gauge
  - name: hrStorageAllocationUnits
    oid: 1.3.6.1.2.1.25.2.3.1.4
    type: gauge
    help: The size, in bytes, of the data objects allocated from this pool - 1.3.6.1.2.1.25.2.3.1.4
    indexes:
    - labelname: hrStorageIndex
      type: gauge
  - name: hrStorageSize
    oid: 1.3.6.1.2.1.25.2.3.1.5
    type: gauge
    help: The size of the storage represented by this entry, in units of hrStorageAllocationUnits
      - 1.3.6.1.2.1.25.2.3.1.5
    indexes:
    - labelname: hrStorageIndex
      type: gauge
  - name: hrStorageUsed
    oid: 1.3.6.1.2.1.25.2.3.1.6
    type: gauge
    help: The amount of the storage represented by this entry that is allocated, in
      units of hrStorageAllocationUnits. - 1.3.6.1.2.1.25.2.3.1.6
    indexes:
    - labelname: hrStorageIndex
      type: gauge
  - name: hrProcessorFrwID
    oid: 1.3.6.1.2.1.25.3.3.1.1
    type: OctetString
    help: The product ID of the firmware associated with the processor. - 1.3.6.1.2.1.25.3.3.1.1
    indexes:
    - labelname: hrDeviceIndex
      type: gauge
  - name: hrProcessorLoad
    oid: 1.3.6.1.2.1.25.3.3.1.2
    type: gauge
    help: The average, over the last minute, of the percentage of time that this processor
      was not idle - 1.3.6.1.2.1.25.3.3.1.2
    indexes:
    - labelname: hrDeviceIndex
      type: gauge
  - name: ssCpuIdle
    oid: 1.3.6.1.4.1.2021.11.11
    type: gauge
    help: The percentage of processor time spent idle, calculated over the last minute
      - 1.3.6.1.4.1.2021.11.11
  - name: memTotalFree
    oid: 1.3.6.1.4.1.2021.4.11
    type: gauge
    help: The total amount of memory free or available for use on this host - 1.3.6.1.4.1.2021.4.11
  - name: memTotalReal
    oid: 1.3.6.1.4.1.2021.4.5
    type: gauge
    help: The total amount of real/physical memory installed on this host. - 1.3.6.1.4.1.2021.4.5
  - name: memAvailReal
    oid: 1.3.6.1.4.1.2021.4.6
    type: gauge
    help: The amount of real/physical memory currently unused or available. - 1.3.6.1.4.1.2021.4.6
  auth:
    community: public
```

- g. Save the file
- h. Workloads > Deployments > snmp-exporter > Details
- i. Recycle the Pod: Decrease the pod count to 0 > Increase the pod count to 1
- j. Confirm success

### Step 21

Update SNMP Exporter Deployment 

- a. Switch from "Developer" to "Administrator" Perspective
- b. Workloads > Deployments
- c. Project > **cid001287-cloud-analytics**
- d. snmp-exporter
- d. Decrease the pod count to 0 
- e. YAML 
- f. Go to section: `spec` > `template` > `spec` > `containers`
- g. Add the following:

```
          args:
            - '--config.file=/snmp-data/snmp.yml'
```

- g. Save
- h. Details > Increase the pod count to 1
- i. Confirm success

### Step 22 

Verify SNMP Exporter is up and running in the customer namespace

- a. Switch from "Developer" to "Administrator" Perspective 
- b. Networking > Routes
- c. Project > **cid001287-cloud-analytics**
- d. Name: **snmp-exporter** > Location > http://snmp-exporter-cid001287-cloud-analytics.apps-priv.atl-stg-ocp-01.cl.sec.ibm.com/
- e. Verify SNMP Exporter UI is 200 Ok. 
- f. Verify SNMP Exporter Route URL (Step 22d) + `/config` response matches Step 19f. 

### Step 23

Add Prometheus Configuration 
(_For SNMP Exporter as a target, targetting an SNMP device to scrape_)

- a. Switch from "Developer" to "Administrator" Perspective
- b. Workloads > Pods
- c. Project > **cid001287-cloud-analytics**
- d. prometheus > Terminal 
- e. `cd /prometheus-data`
- f. Create the file `prometheus.yml` (See Note 1)

```
vi prometheus.yml

# global config
global:
  scrape_interval:     60s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 60s # Evaluate rules every 15 seconds. The default is every 1 minute.
scrape_configs:
  - job_name: 'snmp'
    static_configs:
      - targets:
        - {host-ip}  # The SNMP device to scrape (e.g., 206.253.238.53)
    metrics_path: /snmp
    params:
      module: [default]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: snmp-exporter-cid001287-cloud-analytics.apps-priv.atl-stg-ocp-01.cl.sec.ibm.com:80 # (From Step 21d)

```
- g. Save the file `:wq`
- h. Workloads > Deployments > prometheus > Details
- i. Recycle the Pod: Decrease pod count to 0 > Increase pod count to 1
- j. Confirm success

### Step 24

Install the SNMP Node Exporter Full Dashboard into Grafana

- a. Repeat Step 4
- b. Dashboards Manage > Import > Dashboard ID: `TBD` > Load 
- c. Options > Name: Node Exporter Full
- d. Prometheus > Select a Data Source > Prometheus (default)
- e. Import
- f. Verify Full Dashboard is displayed 200 Ok. 



## Off-Boarding Manual Procedure

### Steps

- a. Switch from "Developer" to "Administrator" Perspective
- b. Project > **cid001287-cloud-analytics**
- c. Workloads > Deployments > Delete Grafana, Grafana Image Renderer, Prometheus, and SNMP Exporter Deployments
- d. Networking > Services > Delete All
- e. Networking > Routes > Delete All
- f. Storage > Persistent Volume Claims > Delete All
- g. Home > Projects > **cid001287-cloud-analytics** > Actions > Delete Project

<!-- Do not edit -->
<hr/>
<footer>
<span>Page Last Updated: {docsify-updated}</span>
</footer>