# Cloud Analytics

## Description

The Cloud Analytics architecture integrates Grafana and Prometheus OSS into the X-Force Protection Platform to bring customer device hardware and OS metrics into a complete monitoring and data visualization solution. The architecture also visualizes event streams from major cloud providers through Grafana data sources for AWS CloudWatch, Azure Insights, and Google Monitoring API. 

## Project Details

* Tribe(s): Data & Identity, Cloud
* Architect(s): Alex Rodriguez, Chris Di Dato
* Product Owner: John Meesseman
* Scrum Master(s): Grażyna Opulska, Gangadhishwar Akkimaradi
* Jira Epic(s): [XPS-60996](https://jira.sec.ibm.com/browse/XPS-60996)

## Project Context

The primary goal of an architecture for Cloud Analytics in the XPP is to provide integrated dashboarding and data visualization capabilities into customer-facing web applications, by leveraging best-in-class open-source software. 

Cloud Analytics in the XPP is intended to serve, more effectively, a segment of customers who derive high value from real-time metrics and health dashboards about devices under management by MSS. The data acquisition and visualization services in the Cloud Analytics sub-system, expose real-time device health metrics to XPP web applications, through visually rich graphs, in a way that can be consumed directly and securely by customer users. 

These features are expected to differentiate and to provide competitive advantage to IBM when Security Offerings introduce user experiences that include interactive visualization of real-time device health metrics. 

#### Journey to Cloud

As the technologies, platforms, software, and vendors that come under management by MSS define roadmaps for their products that take them from on-prem deployment models to SaaS models, hosted by one or more major cloud providers, the Cloud Analytics architecture <i>opens</i> the X-Force Protection Platform to cloud-based monitoring of Amazon Web Services (AWS), Microsoft Azure, and Google Cloud Platform (GCP) hosted customer infrastructure, through an out-of-box integration of Grafana with cloud provider monitoring APIs. 

#### Reduce Costs

The integration of the open-source software into the platform is also expected to completely eliminate front-end web application development costs associated with health metrics visualization and dashboarding, as Grafana provides a vast gallery of dashboards and visualizations for data analytics that can be leveraged easily through point-and-click.

### 1. Features & Functionality

The Cloud Analytics architecture is intended to support the following main features and functionality:

#### A. Data Collection 

Time series collection happens via a pull model from managed endpoints over HTTPS. Targets (endpoints) for data collection are discovered via service discovery or by a static configuration. 

#### B. Cloud Monitoring 

An out-of-box integration of Grafana with cloud provider monitoring APIs provides cloud-based monitoring of Amazon Web Services (AWS), Microsoft Azure, and Google Cloud Platform (GCP) hosted customer infrastructure. 

<b>Note</b>: Cloud monitoring does not result in data acquisition by the XPP, as health metrics for cloud-hosted infrastructure are generated and stored by the cloud provider. 

#### C. Real-time data visualization

Grafana turns time-series database (TSDB) data into graphs and visualizations. 

#### D. Graph Embedding 

The architecture introduces a step into a development workflow for XPP web applications that embeds Grafana dashboard elements (panels), directly into the application, as an interactive UI element that has a connection to a data source and is updated dynamically or in real-time with the state of a managed device. 

### 2. Design & Architecture Methodology 

The design methodology for Cloud Analytics combines aspects of the IBM Garage Method which is more cloud-centric. Design Thinking is used inside the Garage Method. The Garage method combines:

- Design Thinking
- Lean Startup
- Agile Methodology 

Design Thinking - instead of writing stacks of documents, the architecture picks and chooses UMF (Unified Modeling Framework) Work Products to write as needed:

- Architecture Overview
- System Context
- Architectural Decisions
- Component Model
- Threat Model

These UMF Work Products are combined here into a single document, the XPP Platform Architecture Document.  

<hr/>
<footer>
<span>Page Last Updated: {docsify-updated}</span>
</footer>
