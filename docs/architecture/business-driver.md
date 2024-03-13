# Business Driver

## Pricing Model

The following pricing model is intended to illustrate potential monthly billing for customers of Cloud Analytics Services (CAS). Customers may be charged monthly based upon calculated use of the following services:

A. Data Acquisition  
B. Health Monitoring  
C. Alerting & Notification (TBD)

### A. Data Acquisition

The following parameters are factored into monthly data acquisition pricing:

1) Number of health metrics ingested 
2) Data transferred 
3) UV (Unit Value) as a multiple of a daily scrape rate

Health metrics ingested from a device fall into 2 categories: 

- I) Basic Health Metrics
- II) Advanced Health Metrics

#### I. Basic Health Metrics

Basic health for each of the CPU, Memory, and Disk on a device requires 2 metrics to be ingested:

| Observed | Metric 1 | Metric 2| Metric Count
|----|----|----|----
| CPU | CPU Cores | Overall CPU Busy (seconds) | 2
| Memory| Total Memory Available | Total Memory Free | 2
| Disk | Total Disk Available | Total Disk Free | 2
| **Total** | | | **6**

**Basic Health Monitoring Cost Calculation**

Taken together, for basic health, a total of 6 metrics are acquired, where each metric transfers 100 bytes of Prometheus metadata and takes 1 scrape to be ingested. Data _transfer_ price is calculated as total bytes transferred and costs $0.001 per 100 bytes, (1/1000th) of a dollar. A single sample from a Prometheus time-series is estimated to be 100 bytes of data including meta-data or labels. Data ingestion price is calculated per scrape and costs $0.02 per scrape. Data _ingestion_ focuses on the computational resources used to establish authenticated connections to devices, to stream encrypted telemetry data securely over the network, and to write time-series samples into a multi-dimensional, customer specific Prometheus database. 

**Cost Breakdown**

|Metric Count | Data Transfer Cost | Ingestion Cost
|----|----|----
|6 |Metric Count * 100 bytes * $0.001 |Metric Count * Number of Scrapes * $0.02

**Estimated Cost per Month (per Device monitored)**

|Charge |Unit Value (UV) | Scrape Rate | Formula | Monthly Cost | 
|----|----|----|----|----
|Data Transfer | $0.001 | 1,440 per day (one scrape per minute) | 6 * 100 * UV * Rate * 30 days / 100 | $252
|Data Ingestion | $0.02 | 1,440 per day | 6 * 1 * UV * Rate * 30 days / 100 | $51.84
|Data Storage† |  $0.00 | |  | $0.00
|**Total** | | | | **$303.84**

**†Note:** CAS does not charge for storage. Storage of time-series data is short term and rotates on a 30-day basis. By default, the most recent 30 days of data is stored. Long term storage of health metrics for historic reporting, advanced analytics, and for machine learning (ML) purposes will be offered in the future and priced separately. 

#### II. Advanced Health Metrics

Advanced health monitoring acquires deeper metrics from a device to provide a higher level of insight and to prepare time-series data for more meaningful analytics to be applied. 

Advanced health monitoring for the Network Traffic, Time Sync, System Processes, Advanced CPU, Advanced Memory, and Advanced  Disk on a device requires approximately 86 metrics to be ingested:

|Observed | Metric 1 | Metric 2 | Metric Count | Metric Multiple | Metrics Total
|----|----|----|----|----|----
|Network Traffic | Transmit Bytes Total | Receive Bytes Total | 2 | Network Interfaces | 4
|Time Sync Drift | Estimated Time Offset | Time Sync Status | 2 |  | 2
|System Process | Process Blocked | Process Running | 2 | | 2
|CPU Advanced | CPU Seconds | | 1 | CPU Modes <sup>1</sup> * CPU-core | 64
|Memory Advanced | Memory Bytes | | 1 | Memory stack <sup>2</sup> | 9
|Disk Advanced | Filesystem Size| | 1 | Filesystem Mountpoint <sup>3</sup> | 5
| **Total†** | | | | | **86**

† Machine Class: 8-Core CPU, 2 x Network Interface (eth0/eth1)

<sup>1</sup>CPU Modes: a) Idle b) IO Wait c) IRQ d) Nice e) Soft IRQ f) Steal g) System h) User

<sup>2</sup>Memory Stack: a) Apps b) PageTables c) SwapCache d) Slab e) Cache f) Buffers g) Unused h) Swap i) Hardware Corrupted

<sup>3</sup>Mountpoints: a) / b) /var c) /opt d) /boot e) /home 

**Cost Breakdown**

|Metric Count | Data Transfer Price | Ingestion Price
|----|----|----
|86 |Metric Count * 100 bytes * $0.001 |Metric Count * Number of Scrapes * $0.02

**Estimated Cost per Month (per Device monitored)**

|Charge |Unit Value (UV) | Scrape Rate | Formula | Monthly Price | 
|----|----|----|----|----
|Data Transfer | $0.001 | 1,440 per day (1 scrape per minute) | 86 * 100 * UV * Rate * 30 days / 100 | $3,612
|Data Ingestion | $0.02 | 1,440 per day | 86 * 1 * UV * Rate * 30 days / 100 | $722.40
|Data Storage† |  $0.00 | |  | $0.00
|**Total** | | | | **$4,334.40**

### B. Health Monitoring

Health Monitoring pricing is based upon a Unit Value (UV) associated with Prometheus query execution time. This is a calculated cost to retrieve time-series data for monitoring purposes and the compute resources that are allocated to queries. The following parameters are factored into monthly health monitoring pricing:

1. Number of users monitoring a dashboard
2. Average time spent using a dashboard
3. Total number of PromQL queries executed per day (one query per dashboard panel)

Unit Value: There is a $0.165 charge per minute of PromQL execution time. 

**Average Usage**

|Users|Dashboards|Panels| Average Monitoring Time
|----|----|----|----
|1 |1 |3 | 1 Hour per day or 30 Hours per month

**Daily Usage**

| PromQL Queries | Execution Time | Hourly Charge | Daily Charge
|----|----|----|---
| 60 Queries per hour (1 per minute) * 3 panels per dashboard | 180 Queries * 20ms execution time | (3,600 / 1000) <sup>1</sup> * $0.165 | $0.99

<sup>1</sup>**Note:** A total of 3.6 seconds (3,600 / 1000) are allocated to PromQL query execution in this model. 

**Monthly Charge**

| Days in Month | Daily Charge | Monthly Total
|----|----|----
| 30 | $0.99 | **$29.70**

## Pricing Example

Note: The values in a pricing example are intended to be used for price modeling purposes only. 

### Example 1 - Basic Health Monitoring

* Customer: Acme, Inc.
* Integrated Platform: CyberArk
* Devices in the Solution: 4

Acme Inc., an MSS Customer, recently became a PAM customer running the CyberArk solution on-prem. Acme also plans to become a Cloud Analytics Services (CAS) customer for IBM to provide basic health monitoring and device health dashboards in Horizon for the devices in the CyberArk solution. 

Acme expects 5 active users to logon to Horizon on a daily basis to check the Platform Health and Device Health dashboards. 

**A. Data Acquisition**

| Devices Monitored | Monthly Charge per Device | Total Yearly
|----|----|----
| 4 | $303.84 | $14,584.32

**B. Health Monitoring**

| Users | Dashboards | Monthly Charge | Total Yearly 
|----|----|----|----
| 5 | 1 | $29.70 * 5 = $148.50 | $1,782.00

**C. Total CAS Services**

| CAS Service | Total Yearly
|----|----
| Data Acquisition | $14,584.32
| Health Monitoring |  $1,782
| **Total** | **$16,366.32**

## Revenue Estimation 

- As a % percentage of the MSS customer base is boarded into Cloud Analytics Services (CAS), estimate total yearly revenue. 
- For example, if 18,000 devices come under basic health monitoring, at an average cost of $4,091 per device per year (based upon Example 1), then an estimated total yearly revenue for CAS would be **$73.63M**. 
- This does not include an estimate for long-term time-series data storage or Alerting & Notification services. 

## Amazon Pricing

For comparison:

Amazon Managed Service for Prometheus pricing

https://aws.amazon.com/prometheus/pricing/

Amazon Managed Service for Grafana pricing

https://aws.amazon.com/grafana/pricing/
