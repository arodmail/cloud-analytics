
# Project Plan

## Epics

- [XPS-60996 - Platform Health Dashboard - Beta Release](https://jira.sec.ibm.com/browse/XPS-60996)
- [XPS-78026 - Platform Health Dashboard - v1.0](https://jira.sec.ibm.com/browse/XPS-78026)

## Cloud Analytics Roadmap

### Design & Architecture Q4 (Oct-Dec)

**Goal:** Design and architect a complete monitoring and data visualization solution for customer devices/assets on-prem/cloud hosted. Create a prototype/POC. 

1. Create architecture documentation, overview, system context, threat model, etc., first cut. <- COMPLETED
2. Present overall Cloud Analytics solution to MSS: Offering Management, Architecture & Engineering <- COMPLETED
3. Build a Lab hosting the basic components of the architecture <- COMPLETED
4. [XPS-60997](https://jira.sec.ibm.com/browse/XPS-60997) - Prototyping and POC work, backend + frontend <- COMPLETED

### Beta 0.9 - Q1 (Jan-Mar)

#### Goals & Features

A. Operationalize the basic components of the architecture. </br>
B. Monitor hardware metrics on Windows and Linux Devices where the Prometheus Node Exporter can be installed/run. </br>
C. Deploy Grafana/Prometheus into OpenShift. Install/run Node Exporter on PAM Devices (SS Windows lab). </br>
D. Develop & integrate a single device health dashboard in Horizon UI, based on [UX mockups](developer/mockups). </br>

#### Stories to deliver Beta features (beta release):

||Jira ID| Summary | Status
|-----|-----|-----|-----
|1 | [XPS-71818](https://jira.sec.ibm.com/browse/XPS-71818) | Grafana Panel Image Rendering - Perf/Scale | Complete
|2 | [XPS-71821](https://jira.sec.ibm.com/browse/XPS-71821) | Health Dashboard Device Details Tabs - Dev UI | Complete
|3 | [XPS-73261](https://jira.sec.ibm.com/browse/XPS-73261) | Horizon & Grafana Gateway Integration Strategy | Complete
|4 | [XPS-73266](https://jira.sec.ibm.com/browse/XPS-73266) | Install/Deploy Windows Node Exporter | Complete
|5 | [XPS-77018](https://jira.sec.ibm.com/browse/XPS-77018) | SNMP Exporter Config in OpenShift | Complete
|6 | [XPS-76972](https://jira.sec.ibm.com/browse/XPS-76972) | Platform Page Update | Continued
|7 | [XPS-76973](https://jira.sec.ibm.com/browse/XPS-76973) | Device Page Update | Continued
|8 | [XPS-73260](https://jira.sec.ibm.com/browse/XPS-73260) | Grafana Gateway API Design + Implementation | Continued
|9 | [XPS-73267](https://jira.sec.ibm.com/browse/XPS-76787) | Integrate Horizon + Microservices APIs (Device Log Source) | Continued
|10 | [XPS-78487](https://jira.sec.ibm.com/browse/XPS-78487) | Update Device Log Source API | Continued
|11 | [XPS-73259](https://jira.sec.ibm.com/browse/XPS-73259) | Integrate Horizon + Microservices APIs (Device, Ticket) | Continued
|12 | [XPS-77262](https://jira.sec.ibm.com/browse/XPS-77262) | Require more fields from /devices/{id} backend call | Continued
|13 | [XPS-77266](https://jira.sec.ibm.com/browse/XPS-77266) | Require more fields from /requests/{id} backend call | Continued
|14| [XPS-73263](https://jira.sec.ibm.com/browse/XPS-73263) | Prometheus Config in OpenShift | Resolved
|15 | [XPS-73267](https://jira.sec.ibm.com/browse/XPS-73267) | QA Tests -> QX-* Test Cases in Jira | Continued

### Version 1.0 - Q2 (Apr-Jun)

#### Goals & Features

A. Complete Beta features/components  
B. Provide Health Dashboard Platform Page  
C. Integrate data acquired over SNMPv3  
D. End-to-end test for production readiness  

#### Stories to deliver 1.0 features (production release):

||Jira ID| Summary | Status
|-----|-----|-----|-----
|1 | [XPS-76972](https://jira.sec.ibm.com/browse/XPS-76972) | Platform Page Update | Complete
|2 | [XPS-76973](https://jira.sec.ibm.com/browse/XPS-76973) | Device Page Update | Complete
|3 | [XPS-73260](https://jira.sec.ibm.com/browse/XPS-73260) | Grafana Gateway API Design + Implementation | Paused
|4 | [XPS-73267](https://jira.sec.ibm.com/browse/XPS-76787) | Integrate Horizon + Microservices APIs (Device Log Source) | Paused
|5 | [XPS-78487](https://jira.sec.ibm.com/browse/XPS-78487) | Update Device Log Source API | Paused
|6 | [XPS-73259](https://jira.sec.ibm.com/browse/XPS-73259) | Integrate Horizon + Microservices APIs (Device, Ticket) | Blocked
|7 | [XPS-77262](https://jira.sec.ibm.com/browse/XPS-77262) | Require more fields from /devices/{id} backend call | Review
|8 | [XPS-77266](https://jira.sec.ibm.com/browse/XPS-77266) | Require more fields from /requests/{id} backend call | Blocked
|9 | [XPS-73267](https://jira.sec.ibm.com/browse/XPS-73267) | QA Tests -> QX-* Test Cases in Jira | Not Started
|10 | [XPS-80377](https://jira.sec.ibm.com/browse/XPS-80377) | Device Logs Info API Design/Implement | Formulated
|11 | [XPS-80378](https://jira.sec.ibm.com/browse/XPS-80378) | Integrate Horizon + Microservices APIs (Device Log Info) | Formulated
|12 | [XPS-80392](https://jira.sec.ibm.com/browse/XPS-80392) | Platform Health Dashboard Ticket Counts | Formulated
|12 | [XPS-TBD]() | Platform Page Health Monitored Panel Design | Design
|13 | [XPS-TBD]() | OpenShift Cluster to Monitored Customer Device Network Access | Design

### Version 1.1 - Q3 (Jul-Sept)

#### Goals & Features

A. Alerting & Notification  
B. Automate on-boarding of customer device dashboarding + monitoring services  
C. Nagios/Prometheus/Grafana Integration  
D. Integrate data acquired over SSH  

#### Stories to deliver 1.1 features:

||Jira ID| Summary | Status
|-----|-----|-----|-----
|1 | [XPS-TBD]() | OpenShift Deployment Automation - (Automate [On-Boarding Manual Procedure](architecture/operational-management?id=on-boarding-manual-procedure)) | Design
|2 | [XPS-TBD]() | SSH Exporter Config in OpenShift | Story Formulation

### Version 2.0 - Q4 (Oct-Dec)

A. Visualize event streams from cloud providers   
B. Expand data acquisition across Platforms (TBD)  
C. Retain Long Term Data: InfluxDB, OpenTSDB, and/or Graphite  

1. Monitoring Cloud Assets -> AWS CloudWatch  
2. Monitoring Cloud Assets -> Azure Monitor  
3. Monitoring Cloud Assets -> GCP Cloud Monitoring  

<!-- Do not edit -->
<hr/>
<footer>
<span>Page Last Updated: {docsify-updated}</span>
</footer>
