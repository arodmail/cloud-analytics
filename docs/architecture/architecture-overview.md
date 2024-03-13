# Architecture Overview

### Hosted Grafana

<img src="_media/hosted-grafana.png" alt="Hosted Grafana" style="display: block; margin-left: auto; margin-right: auto; height: 80%; width: 80%; padding-top: 25px; padding-bottom: 25px;"/>

- Grafana as a control plane management service
- Run Grafana internally in a container orchestrator as an integral sub-system of the XPP, authenticate to time-series data sources
- Define or import dashboards and panels to visualize hardware metrics, cpu, memory, disk, i/o, etc. 
- Embed png rendered Grafana panels in Web Applications, or refresh/reload in-place interactively through iframes
- Visualize normalized telemetry data from cloud-based data sources and time-series data from cloud assets
- A modern alternative to traditional forms of monitoring (i.e, Nagios/Icinga + NRPE + SNMP)

### Multi-Cloud 

<img src="_media/multi-cloud.png" alt="Multi-cloud" style="display: block; margin-left: auto; margin-right: auto; height: 80%; width: 80%; padding-top: 25px; padding-bottom: 25px;"/>

- Visualize event streams from cloud providers
- Light on development/engineering for Grafana to be the interface to all cloud monitoring
- For example: by default, in AWS, Lambda functions produce a CloudWatch log stream and standard metrics.
- Tradeoffs: May be somewhat heavy on bandwidth, but very light on storage

### Basic Elements of the Architecture

<img src="_media/elements.png" alt="Basic Elements" style="display: block; margin-left: auto; margin-right: auto; height: 80%; width: 80%; padding-top: 25px; padding-bottom: 25px;"/>

1. <b>Endpoints:</b> span a very broad range of hardware and software across a wide spectrum of device types. See [Exporters & Integrations](https://prometheus.io/docs/instrumenting/exporters/) for a complete list. 
2. <b>Time-series Database:</b> Prometheus is a free, OSS time-series database developed as a alternative to StatsD and Graphite based solutions. Built to provide a multi-dimensional data model, operational simplicity, scalable data collection, and a powerful query language, PromQL.
3. <b>Dashboarding Tool:</b> Grafana is OSS, used as visualization and analytics software. Provides metrics visualization for time-series data sources, runs on multiple platforms, provides a complete monitoring solution. 
4. <b>Web Applications:</b> "consumers" of Grafana dashboards and panels, apps embed device health metrics visualizations, tables, single stats, and other forms of data analytics provided by Grafana. The solution is intended to significantly reduce or to completely eliminate engineering costs associated with in-house development of metrics visualization dashboarding capabilities. 
5. <b>Permissions Logic / APIs:</b> to prevent un-authorized access to Device data, a microservices layer is integrated into the architecture.  

#### Pull Model

- The Prometheus server collects data and stores it in its time-series database
- Prometheus is based upon a “pull” and not a “push” model
- Prometheus server reaches out and pulls metrics from all endpoints on a "scrape interval". 

### Multi-Tenancy

<img src="_media/multi-tenant.png" alt="Multi Tenancy" style="display: block; margin-left: auto; margin-right: auto; height: 80%; width: 80%; padding-top: 25px; padding-bottom: 25px;"/>

- A customer is associated with a Grafana instance through a url scheme. 
- Isolation between data for different customers (tenants) is achieved through the deployment and operationalization of customer-specific Prometheus databases and customer-specific Grafana hosts. 
- Prometheus and Grafana run in separate containers, one per customer, or scaled horizontally to meet increased performance needs. 
- Access control for device data, is provided by a sidecar attached to a Device API microservice. The sidecar returns only customer authorized data to authorized customer users, based upon customer permissions. 
- A customer's Grafana url and/or device dashboard location is afforded the same level of protection as a device IP. 
- Applications sending requests to a customer-specific hosted Grafana are expected to be designed well, and never to include user provided parameters in urls that send requests to Grafana. 
- See [Threat Model](/architecture/threat-model) for more details about customer data isolation, network zones, IDS/IPS, firewall, assets, threats, and controls. 

### Logical Isolation per Tenant

<img src="_media/logical-isolation.png" alt="Logical Isolation" style="display: block; margin-left: auto; margin-right: auto; height: 60%; width: 60%; padding-top: 25px; padding-bottom: 25px;"/>

- Intra-Cluster Multi-Tenancy – SaaS Model
- One namespace per Customer
- Per-Customer isolation = Namespaces x 2,500 (potentially)
- SaaS = after the app is deployed, consumers connect directly to the app
- Consumers are applications: Horizon, Glass, or other web app
- Namespaces provide logical isolation between tenants (customers) 
- K8s resources are namespace-scoped
- Shared control plane means Ops team can audit, operate, monitor, all tenants (customers)
- Single workload = a K8s Deployment 
- Optionally use K8s Policies (namespace-scoped) and IAM to authenticate/authorize access by users to namespace resources

#### Alternatives

- Enterprise-model: suitable model for users who belong to same organization/company
- Cluster per tenant: pros: strong network isolation, cons: slow boarding/turn-up + operational complexity

### Persistent Storage

To maintain state in between application restarts, NFS and/or NAS is used as a mounted storage volume into the container orchestrator used to host containerized versions of the Grafana and Prometheus applications. 

<img src="_media/persistence.png" alt="Persistent Storage" style="display: block; margin-left: auto; margin-right: auto; height: 70%; width: 70%; padding-top: 25px; padding-bottom: 25px;"/>

- OpenShift Container Platform clusters can be provisioned with persistent storage using NFS. 
- Persistent volumes (PVs) and persistent volume claims (PVCs) provide a convenient method for sharing a volume across a project.

### Alerting & Notification

<img src="_media/alerting-notification-arch.png" alt="Alerting & Notification" style="display: block; margin-left: auto; margin-right: auto; height: 80%; width: 80%; padding-top: 25px; padding-bottom: 25px;"/>

**Sample Flow**

- Grafana Dashboard Alerts are triggered by defined thresholds
- A Grafana alert notification is sent through a channel (e.g., Slack or a Web hook, an HTTP request to any URL)
- A microservice listens for incoming Grafana alerts to translate into an ITSM ticket (or service request)
- An Analyst is notified via an ITSM to respond to an alert through a ticket


### Authentication

A simple authentication approach by Grafana into a cloud-hosed asset is possible through Basic Authentication with a cloud issued access key/secret:

<img src="_media/data-source-auth.png" alt="Data Source Authentication" style="display: block; margin-left: auto; margin-right: auto; height: 80%; width: 80%; padding-top: 25px; padding-bottom: 25px;"/>

- Create User(s) in an AWS account
- User belongs to a Role
- Policy is attached to Role
- Policy grants ListMetrics and GetStatistics permissions to the Role
- User is issued security credentials: an Access Key + Access Secret
- Credentials are used by Grafana to authenticate to the CloudWatch Data Source

### SAML Federation

In a single Grafana instance deployment model, multiple tenants are housed entirely in one Grafana server/host. This model requires federated identity and cloud-based IAM. 

<img src="_media/federated.png" alt="SAML Federation" style="display: block; margin-left: auto; margin-right: auto; height: 80%; width: 80%; padding-top: 25px; padding-bottom: 25px;"/>

- SAML authentication allows Grafana users to log in by using an external SAML 2.0 Identity Provider (IdP). 
- Grafana becomes a Service Provider (SP) in the authentication flow, interacting with the IdP to exchange user information.
- Available in Grafana Enterprise v6.3+
- Note: this model is subject to review and validation. 

