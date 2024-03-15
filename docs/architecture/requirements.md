# Requirements

## Functional

> Specify the *capabilities* needed by users of the system to fulfill their job. Functional requirements answer  the question of *what* the solution's sponsor needs, but not *how* it is achieved.

#### 1. Data Collection 

Software engineers, device management personnel, and operations staff should be able to install, configure, run, and monitor a component that collects hardware, software, and operating system metrics from a managed device. 

#### 2. Cloud Monitoring 

Software engineers, device management personnel, and operations staff should be able to integrate the Cloud Analytics sub-system with cloud provider monitoring APIs to monitor cloud-hosted customer infrastructure. 

#### 3. Real-time data visualization

Software engineers, device management personnel, and operations staff should be able to import, customize, and/or create visualizations of time-series database (TSDB) data. 

#### 4. Graph Embedding 

Software engineers should be able to embed dashboards, dashboard elements (e.g., panels), into a web application or mobile application, as an interactive UI element that has a connection to a data source that is updated dynamically or in real-time with the state of a managed device or cloud-hosted customer asset. 

<!-- Do not edit -->
<hr/>
<footer>
<span>Page Last Updated: {docsify-updated}</span>
</footer>

## Non-Functional

> This artifact is used to describe the *quality attributes of the system* and the *constraints* which the design options will be required to satisfy in order to deliver the business goals, objectives or capabilities. It is used to negotiate between, and select from, competing design options. It is used to assess the sizing, cost, and viability of the proposed system. It is a key consideration for understanding service level requirements for operational management of the solution. For more information see the [Non-Functional Requirements](https://ibm.biz/BdzYnV) work product from the UMF. 

A Cloud Analytics sub-system for the X-Force Protection Platform is designed to meet the following non-functional requirements (NFR): 

| ID | NFR | Description | Criteria |
| --------  | ----------------- | --------- | ----------- |
| 1. | Scalability | Scale horizontally | The system must be able to perform well under increasingly higher workloads by adding more nodes to support growth in the number of customer users as well as growth in the amount and variety of data collected from endpoints over time. |  
| 2. | Data Retention | Store data for short and long term customer needs | The system should provide data storage capacity for short term metrics visualization (multi-week), and for long term historic analysis (multi-month). |  
| 3. | Operability | Operate simply and reliably | The system should be built and deployed to operate successfully without needing regular application restarts or server initialization. Operability aims to create a simpler system to operate and maintain, with a reduced cost of ownership, and fewer operational issues. |  
| 4. | Portability | Run anywhere. Export dashboards, graphs, and stats | The system must be designed to run in different environments without any machine dependent installation requirements. The system should also be able to expose and/or export metrics visualization to web and mobile applications in a portable, cross-device manner. |  
| 5. | Aggregability | Aggregate from multiple data sources for analysis | The system must provide a mechanism to collect metrics from a broad range of customer devices, assets, and infrastructure under management by MSS in a way that can be combined into aggregate data suitable to create summary statistics. |  
| 6. | Interoperability | Interoperate with other monitoring systems | The system should be designed to exchange (bring in or share) time-series data with other monitoring systems |  
| 7. | Standardized | Align with an emerging, open standard, for example, [OpenMetrics](https://github.com/OpenObservability/OpenMetrics) | The system may collect and expose data in a non-proprietary, standard format that is defined by a specification maintained by a standards organization. Ideally, the format used by the system should not be encumbered by any copyrights, patents, trademarks or other restrictions. |  
| 8. | Isolated | Isolate customers from each other | The system must be architected to serve multiple customers while isolating customer users from each other, such that users who belong to different customers do not share or see each other's data. Isolation in the system should also shield customers operationally from a single failure in one customer's instance, due to a sudden increased workload or traffic spike in another. |  
| 9. | Security | Protect the system from intrusion and unauthorized use | The system must ensure that all data inside the platform, or its part, will be defended from attacks and unauthorized access. |  

<!-- Do not edit -->
<hr/>
<footer>
<span>Page Last Updated: {docsify-updated}</span>
</footer>
