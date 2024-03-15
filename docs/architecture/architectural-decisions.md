# Architectural Decisions

> For more information see [Architectural Decision](https://en.wikipedia.org/wiki/Architectural_decision).

| Decision | Rationale | Implication |
| -------- | --------- | ----------- |
| Integrate Grafana/Prometheus OSS into the XPP as a complete health monitoring solution for customer assets | Traditional forms of device monitoring such as SNMP (has been around since the 80s), NRPE, and SSH are largely incompatible with modern, cloud-based hosting of customer infrastructure that requires monitoring. This technical incompatibility restricts the XPP in the range of health monitoring that it can do, and the restriction impairs business growth. | The integration of Grafana/Prometheus into the XPP opens cloud-based monitoring of AWS, Azure, and GCP hosted infrastructure through an out-of-box integration of the software with cloud providers. The integration of the OSS into the platform also is expected to completely eliminate front-end Web Application development costs associated with health metrics visualization and dashboarding, as Grafana provides a kind of "vast gallery" of dashboards and visualizations for data analytics that can be leveraged easily through point-and-click.  
| Run Grafana/Prometheus Docker images | A containerized version of the Grafana and Prometheus software provides portability across platforms and container orchstrators, uses fewer resources than VMs at runtime, uses host resources more efficiently, and integrates well with SecDevOps CI/CD tooling and processes. | Running Docker images for Grafana and Prometheus means the applications will start up fast, will scale horizontally, and may be hosted directly on bare-metal or on a virtualized host. Deployment and management of the applications also becomes easier as installation, upgrade, and rollback processes are built into container orchestrators. |
| Tenant isolation: provision Grafana/Prometheus into a namespace per customer (tenant). | Namespaces provide logical isolation between tenants, K8s resources are namespace-scoped | Shared control plane means Ops team can audit, operate, monitor, all tenants (customers). In this model, a single workload is a K8s "Deployment" of Grafana, and a second workload is a Deployment of Prometheus. A namespace per tenant also allows K8s Policies (namespace-scoped) to be used, and IAM to authenticate/authorize access by users to namespace resources. |  

<!-- Do not edit -->
<hr/>
<footer>
<span>Page Last Updated: {docsify-updated}</span>
</footer>
