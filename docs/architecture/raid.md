# RAID Log

> The purpose of RAID (**R**isks, **A**ssumptions, **I**ssues, **D**ependencies) is to capture the important concerns of the project and manage them proactively.  Documenting these concerns will allow important considerations from being overlooked.  It also avoids projects being derailed by unexpected situations and helps meet commitments.  The RAID log is described in the [Viability Assessment](https://ibm.biz/BdzY8x) work product in [IBM MethodWeb](https://ibm.biz/BdzYnv).

<BR><BR>

|ID|RAID|Statement|Impact|Affected Components|Actions|
|---------|-------|-------|-------|-------|-------|
| 1 | Dependency | Cloud-based monitoring APIs are not guaranteed to deliver data consistent with IBM SLAs with customers | An outage in a cloud monitoring API may disrupt service until the outage is addressed. | Web Applications that consume from  Cloud Analytics in the XPP | Tailor IBM SLAs according to cloud-provider uptime (e.g., "three-nines", 99.9% available, etc.) |
| 2 | Assumption | The Prometheus Node Exporter component is a primary source of metrics from a managed device. It's possible that some customers may prefer not to host this component on the device itself. Reasons for this customer preference include: the component runs in a process that may compete for system  resources on the device itself, and it opens a port. | In these cases, the solution cannot collect data from these devices using the Node Exporter. | Assumption is that in most cases, this is a non-issue. But an alternate approach is prudent to have available for customers who resist the use of the Node Exporter in the solution. | Action: 1) verify with OMs for PAM+DAMS that hosting Node Exporter on managed devices for these offerings is viable for most, if not all customers. 2) Include details about alternate approach for when use of Node Exporter is not possible. | 
| 3 | High Risk | Any dependency on SNMP as a monitoring solution exposes the XPP and customer managed devices to high risk CVEs. | Potential compromise of customer managed devices to a determined hacker. See [Security Concerns](developer/data-acquisition?id=security-concerns) for more details. | All Guardium devices and any other platforms that rely on SNMP as a monitoring protocol. | Mandatory use of SNMP Version 3 to mitigate the risks in v1 and v2. Recommended security audit of SNMP exposure and risk assessment across the X-Force Protection Platform. 

<!-- Do not edit -->
<hr/>
<footer>
<span>Page Last Updated: {docsify-updated}</span>
</footer>
