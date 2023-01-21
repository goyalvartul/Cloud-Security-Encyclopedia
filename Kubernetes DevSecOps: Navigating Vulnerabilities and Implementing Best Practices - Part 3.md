# Kubernetes DevSecOps: Navigating Vulnerabilities and Implementing Best Practices - Part 3


## Ingestion into Sumo Logic

There is a significant benefit to having all this data come together in one place. Metrics act as the smoke detector, and logs allow us to investigate and identify the root cause of an issue. By unifying these data sources around a common metadata language, we can easily correlate these signals and pivot between the metrics data about a cluster, the events data about a cluster, and the logs data about an application.

Metadata enables us to create a hierarchical view of a cluster. By linking pods to their services or grouping nodes by cluster, it becomes easier to explore the Kubernetes stack. By utilizing the Auto-discovery capabilities inherent in Prometheus, we can ensure that the hierarchy visualized in Sumo Logic is accurate and up to date.

Rich metadata enables Sumo Logic to automate the building of the Explorer hierarchy of the components present in your cluster and keeps the Explorer current as pods are added and removed. This can save a lot of time and effort for your team, as it eliminates the need for manual updates and ensures that the hierarchy remains accurate and up to date at all times. Additionally, it makes it much easier to explore and understand the Kubernetes stack, which can be quite complex and hard to navigate. By having a clear hierarchical view of the cluster, you can quickly identify and troubleshoot any issues that may arise.



![](images/image37.png)

*With rich metadata, Sumo Logic can automatically construct the explorer hierarchy of the components in the cluster and maintain it as new pods are added or removed, this helps to keep the explorer up-to-date and accurate.*


## Tying together DevOps and SecOps
It is essential to consider security during the development process. Code should be designed in a way that logs are informative and useful. Code analysis is crucial, as is unified observability. All teams should have access to the same data. There is still a significant division between these teams. As systems become more distributed, these teams and their data need to come together.

Kubernetes is an excellent example of how teams can work together in distributed environments. With potentially hundreds of microservices running in an application, containerization and organized distributed systems that utilize Kubernetes become essential. Kubernetes is one of the most distributed systems available and its architecture has built-in tooling that makes it easy to collect data from highly federated infrastructure. This makes it ideal for organizations with multiple teams and microservices that need to work together seamlessly. Furthermore, with the increasing adoption of cloud-native technologies, it becomes more critical to have a well-organized and distributed system that can handle the complexity of modern applications. Kubernetes is the perfect solution for this, and its built-in tooling makes it easy to collect, analyze and monitor data from different sources and make it available to all teams.

Dameon sets provide standardization for monitoring across nodes, allowing data to be collected and analyzed using various backends tools.

- To improve investigation, it would be useful for the security team to have access to information about deployment metrics and cluster performance.
- The development team should focus not only on logging output but also on performance metrics when troubleshooting their application in production.
- The ITops team requires observability data for both cloud infrastructure and the apps running on it, to ensure successful deployments and smooth operations.



![](images/image38.png)



*Security information can be accessed at the cluster level, along with log, metric, and event data.*

Your security and development team can enhance the security visibility by providing information about security policies and controls, as well as relevant events, in relation to the Kubernetes framework.
