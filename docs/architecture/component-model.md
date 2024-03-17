# Component Model

> The purpose of the component model is to help organize projects, manage the complexity of the solution, and ensure that all architectural requirements have been addressed. It helps developers design and implement the solution and understand the big picture of the system design. For more information see the [Component Based Software Engineering](https://en.wikipedia.org/wiki/Component-based_software_engineering).

## 1. Grafana Gateway Design

### 1.1 Description

The Grafana Gateway sits between a web application and a collection of Grafana instances for multiple customers. In the Cloud Analytics architecture, each customer is provisioned into an instance of Grafana in order to support custom dashboarding and panels per customer and to separate one customer's data from another in a multi-tenant environment. 

<img src="_media/grafana-gateway-overview.png" alt="Grafana Gateway Overview" style="display: block; margin-left: auto; margin-right: auto; height: 80%; width: 80%; padding-top: 25px; padding-bottom: 25px;"/>

### 1.2 Design Goals

Why use a Grafana Gateway? Grafana is hosted per customer in an OpenShift cluster, modeled as a microservice that exposes dashboarding and metrics visualization APIs. The gateway accepts a request and returns a response with the following design goals for large-scale request processing in mind:

- Protect customer specific Grafana instances from cross-customer malicious use through authentication and authorization. 
- Perform detailed logging for analytics and monitoring purposes.
- A response from a single call for a Grafana dashboard is aggregated from multiple, distinct backend calls.
- As new Grafana instances are added, and others are shut down, applications going through the gateway will still find all Grafana resources in the same place.
- Use a Grafana backend plugin that handles rendering panels to PNGs using a headless browser (Chromium).

### 1.3 Request Processing 

The Grafana Gateway screens all requests for Grafana panels by performing the following steps:

1. Performs Bearer authentication. Parses an authorization Header and verifies a JWT
2. Extracts a Customer ID from a JWT
3. Resolves a Device IP from a Machine Host Name OR resolves a Device IP from a Device ID (using [Device API](https://services.sec.ibm.com/api_explorer/swagger-ui/?url=https://services.sec.ibm.com/api_explorer/swagger/device_ms))
4. Assembles a full URL to a Grafana PNG panel 
5. Sends an HTTP GET request to a customer's Grafana instance for a panel PNG
6. Returns the PNG image for a Grafana panel (Mime Type = image/png)

### 1.4 Sample Request (Single Panel)

```
HTTP GET https://grafana-gateway-cloud-analytics.apps-priv.atl-stg-ocp-01.cl.sec.ibm.com/grafana-panel?panelID=20&deviceID=PR0000000033799
Authorization: Bearer <JWT> (contains a CustomerID claim)
```

### 1.5 Sample Response

```
ResponseCode: 200 OK
Content-Type: image/png
Response:
https://grafana-cid001287-cloud-analytics.apps-priv.atl-stg-ocp-01.cl.sec.ibm.com/render/d-solo/rYdddlPWk/node-exporter-full-cid001287?orgId=1&panelId=20&width=1000&height=500&var-node=206.253.238.53:9100
```

### 1.6 Grafana Image Renderer URL

Requests for panel PNG images sent to the Grafana Gateway are translated into requests to a customer's Grafana instance. For example, a direct link URL to a panel PNG:

https://grafana-cid001287-cloud-analytics.apps-priv.atl-stg-ocp-01.cl.sec.ibm.com/render/d-solo/rYdddlPWk/node-exporter-full-cid001287?orgId=1&panelId=20&width=1000&height=500&var-node=206.253.238.53:9100

### 1.7 Direct Link URL Parts

| URL Part | Sample Value | Description
| ---- | ---- | ---- |
| Scheme | `https://` | Protocol used to access the panel
| Application Name | `grafana` | Name of the Grafana application hosted in the OpenShift cluster
| OpenShift Namespace | `-cid001287-cloud-analytics` | The cloud analytics namespace for customer applications in the OpenShift cluster for Customer ID: **CID001287**
| OpenShift Security Context| `.apps-priv` | OpenShift Security Context contraint (?)
| OpenShift cluster | `.atl-stg-ocp-01.cl` | OpenShift cluster environment (see Table below for dev, stg, prod values)
| Domain | `.sec.ibm.com` | IBM domain for IBM Security 
| Grafana Path | `/render/d-solo/rYdddlPWk/` | Path to Customer Grafana dashboards
| Dashboard Name | `node-exporter-full/` | Name of customer Grafana dashboard 
| Panel ID | `&panelId=20` | Parameter: required, a unique ID for a panel on a dashboard to render
| Width | `&width=1000` | Parameter: optional width for png rendered panel image
| Height | `&height=500` | Parameter: optional height for png rendered panel image
| Node Exporter IP:Port | `&var-node=206.253.238.53:9100` | The monitored device IP where Node Exporter runs and listens on Port 9100. The exporter full dashboard accepts `var-node` as a parameter to switch between monitored hosts using the same dashboard. 

#### 1.8 Open Shift Cluster environment

|Environment| Cluster |
|----|----|
| Dev | `dal09-dev-ocp-01.cl`
| Stg |`atl-stg-ocp-01.cl` 
| Prd | `atl-prd-ocp-01.cl`

### 1.9 Grafana Gateway API Specification

The Swagger definition for the Grafana Gateway API should expose the following HTTP Methods:

| Method | Path | Parameter(s) | Response Content-Type
|----|----|----|----|
| GET | `/grafana-panel` | `panelID=20&deviceID=PR0000000033799` | image/png
| GET | `/grafana-panel` | `panelID=20&machineHostName=WIN-K10GK8CVS4` | image/png
| GET | `/grafana-multi-panel` | `panelID=20&panelID=30&deviceID=PR0000000033799` | application/json

#### 1.10 Sample Request (Multi Panel)

```
HTTP GET https://grafana-gateway-cloud-analytics.apps-priv.atl-stg-ocp-01.cl.sec.ibm.com/grafana-multi-panel?panelID=20&panelID=30&deviceID=PR0000000033799
Authorization: Bearer <JWT> (contains a CustomerID claim)
```

#### 1.11 Sample Response (Multi Panel)

```
ResponseCode: 200 OK
Response Body:
{
   "grafana-panel-png-urls":[
      {
         "id":20,
         "url":"https://grafana-gateway-cloud-analytics.apps-priv.atl-stg-ocp-01.cl.sec.ibm.com/render/d-solo/rYdddlPWk/node-exporter-full-cid001287?orgId=1&panelId=20&width=1000&height=500&var-node=206.253.238.53:9100
      },
      {
         "id":30,
         "url":"https://grafana-gateway-cloud-analytics.apps-priv.atl-stg-ocp-01.cl.sec.ibm.com/render/d-solo/rYdddlPWk/node-exporter-full-cid001287?orgId=1&panelId=30&width=1000&height=500&var-node=206.253.238.53:9100"
      }
   ]
}
```

#### 1.12 API Security

In processing an HTTP GET request, the Grafana Gateway should ensure that:

1. An `Authorization` HTTP Header is present
2. The `Authorization` HTTP Header value is `Bearer <JWT>`
3. The `<JWT>` in the HTTP Header value is verified by the [JWT Provider API](https://services.sec.ibm.com/api_explorer/swagger-ui/?url=https://services.sec.ibm.com/api_explorer/swagger/jwt_provider) `/verify` method
4. The Claims returned by the JWT Provider API include a "customerId" claim

#### 1.13 Sample JWT Claims

```
{
  "x-remoteip": "209.134.187.156",
  "sub": "Microservices",
  "iss": "sec.ibm.com",
  "privileged-user": false,
  "exp": 1612763325,
  "username": "qatest",
  "API_NAME": "authentication",
  "customerId: "P000000614"
}
```

If any of 1-4 is not true, then the response should be 401 Unauthorized. 

## 2. Device Log Info API

### 2.1 Description

The Device Log Info API is intended to provide basic information about log ingestion by LMS for customer devices. 

### 2.2 Design Goals

The Device Log Info API is a REST API exposed by a microservice that is responsible for servicing requests for basic information about device logs. The information about device logs is read from an Elastic Search Index. 

### 2.3 Request Processing

In processing a request, the Device Log Info API should:

- Securely store and retrieve credentials used to establish connections to Elastic Search (e.g., in an OpenShift secret)
- Provide a retry mechanism if the connection or a request to ES times-out or fails (retry x 3)
- Be designed as a stateless service (no caching) 
- Provide a GET method to retrieve device log details by Device ID
- Accept a `unitByte` (KB, MB, or GB) as a parameter to return the `sizeOnDisk` bytes translated into the unit byte specified
- Calculate the difference in years between the oldest log date and newest log date to generate a log retention period
- Create `log4j` logs in a traceable manner with sufficient detail for troubleshooting (e.g., session ID included per log)
- Be portable as an implementation without any code that is specific to a host environment (e.g., OpenShift specific code)

### 2.4 Sample Request

```
HTTP GET https://device-log-info-*.sec.ibm.com/device_log_info?deviceID=PR0000000033799&unitByte=MB
Authorization: Bearer <JWT> (contains a CustomerID claim)
```

### 2.5 Sample Response

```
ResponseCode: 200 OK
Response Body:
{
   "deviceId": "S00000007019415",
   "newestLog": "2021-01-30 12:59:59.0",
   "oldestLog": "2020-01-30 12:59:59.0",
   "logRetentionPeriodYears": "1",
   "sizeOnDisk": "100",
   "lastModifiedDate": "2021-01-30 12:59:59.0",
   "unitByte": "MB"
}
```

### 2.6 Device Log Info API Specification

The Swagger definition for the Device Log Info API should expose the following HTTP Method(s):

| Method | Path | Parameter(s) | Response Content-Type
|----|----|----|----|
| GET | `/device_log_info` | `deviceID=PR0000000033799` | application/json
| GET | `/device_log_info` | `deviceID=PR0000000033799&unitByte=MB` | application/json

Note: if a `unitByte` parameter is not specified, then should default to bytes. 

### 2.7 Data Synchronization

The Data Sync API synchronizes data one-way from SQL databases to Elastic Search indexes. The API exposes a POST method to create a Data Sync Job with settings about a SQL database server, port, host, database name, and a source table. A Data Sync Job is custom configured, runs on a daily basis, and does a one-way sync in which all the data from a source table is replicated into an ES index.

<img src="_media/data-sync.png" alt="Data Sync" style="display: block; margin-left: auto; margin-right: auto; height: 80%; width: 80%; padding-top: 25px; padding-bottom: 25px;"/>

Data Sync Jobs that are successfully created are immediately configured to run once daily. The ES index name created as a result of a sync is identical to the name of the source table. The ES document type added to the index is identical to the name of the database and there is a one-to-one mapping between a table row and an ES document. For example, a "device_info" database table in the “mss” database is sync’d into a "device_info" ES index whose document type is "mss". Data in an ES index can be queried using the Elastic Search API.

For the Device Info API, the following Data Sync Job will be defined in the prod environment:

```
{
    "name": "DeviceLogInfoSyncJob",
    "host": "atla-consdb-01a.mss.iss.net",
    "database": "mss",
    "port": "3306",
    "table": "device_info"
  }
```

#### 2.7.1 Database Hosts (for ConsDB)

To define a Data Sync Job for the `device_info` table in each of dev and stg, the following host names may be substituted per environment:

| Environment | Database Host | 
|----|----|
| Dev | dal09-dev-consdb-01a.sec.ibm.com
| Stg | atl-stg-mssdb-01a.mss.iss.net
| Prod | atla-consdb-01a.mss.iss.net

#### 2.7.2 API Explorer

The Data Sync API is available through API Explorer:

| Environment | REST API | 
|----|----|
| Dev | [Data Sync API](https://dal09-dev-services.sec.ibm.com/api_explorer/swagger-ui/?url=https://dal09-dev-services.sec.ibm.com/api_explorer/swagger/data_sync)
| Stg | [Data Sync API](https://stg-services.sec.ibm.com/api_explorer/swagger-ui/?url=https://stg-services.sec.ibm.com/api_explorer/swagger/data_sync)
| Prod | [Data Sync API](https://services.sec.ibm.com/api_explorer/swagger-ui/?url=https://services.sec.ibm.com/api_explorer/swagger/data_sync)

## 3. Horizon Integration Flows

To populate the Device Health Dashboard with data, the following flows integrate Horizon Web Common Services  with XPP microservice endpoints. 

### 3.1 Get Dashboard Panel

<img src="architecture/_media/sequence-grafana-panel.png" alt="Grafana Panel Sequence" style="display: block; margin-left: auto; margin-right: auto; height: 80%; width: 80%; padding-top: 25px; padding-bottom: 25px;"/>

### 3.2 Get Device Details

<img src="architecture/_media/sequence-device-details.png" alt="Device Details Sequence" style="display: block; margin-left: auto; margin-right: auto; height: 80%; width: 80%; padding-top: 25px; padding-bottom: 25px;"/>

### 3.3 Get Device Log Info

<img src="architecture/_media/sequence-device-log-info.png" alt="Device Log Info Sequence" style="display: block; margin-left: auto; margin-right: auto; height: 80%; width: 80%; padding-top: 25px; padding-bottom: 25px;"/>

### 3.4 Get Tickets

<img src="architecture/_media/sequence-tickets.png" alt="Get Tickets Sequence" style="display: block; margin-left: auto; margin-right: auto; height: 80%; width: 80%; padding-top: 25px; padding-bottom: 25px;"/>

### 3.5 Get Applications (Log Sources)

<img src="architecture/_media/sequence-log-sources.png" alt="Log Sources Sequence" style="display: block; margin-left: auto; margin-right: auto; height: 80%; width: 80%; padding-top: 25px; padding-bottom: 25px;"/>

<!-- Do not edit -->
<hr/>
<footer>
<span>Page Last Updated: {docsify-updated}</span>
</footer>
