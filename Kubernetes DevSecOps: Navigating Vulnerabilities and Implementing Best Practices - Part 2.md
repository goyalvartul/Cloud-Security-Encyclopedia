# Kubernetes DevSecOps: Navigating Vulnerabilities and Implementing Best Practices - Part 2


## Kubernetes hardening best practices
Given that Kubernetes' security capabilities are limited, it is essential for teams to implement additional measures to secure their clusters. The best practices for maximizing Kubernetes' security features and utilizing external tools and strategies for added security are listed below.

### **Configure pod security and network policies**
As previously mentioned, security policies can be used to enforce restrictions on pods and networks, but they are not typically enabled by default in most Kubernetes distributions. Even if they are turned on by default, they will likely need to be customized to fit the needs of your team.

It is crucial for your security team to harden the Kubernetes cluster and set up and enforce policies that reflect your team's requirements. The level of strictness for these policies will depend on the level of security needed for the cluster.

For instance, a production cluster will likely have more restrictive policies (such as policies that prohibit write-access to resources and prevent non-essential network traffic) compared to a cluster used for internal development, testing and deployment (where strict security policies are not as crucial since the cluster will not be running critical applications that are connected to the public internet).

### **Kubernetes host security**
The security of Kubernetes is dependent on the security of the operating systems that run its nodes. As Kubernetes is not able to monitor or secure the host operating systems, it is the responsibility of the administrators to ensure that they are secure. This applies to on-premises hosts, whether or not the infrastructure is containerized.

A best practice is to select a host Linux distribution with minimal components, as additional operating system apps or services that are not required for Kubernetes can increase the attack surface unnecessarily. Another option is to use a bare-metal deployment setup, in which no operating system is used at all (such as in IoT systems). It is also recommended to enable SELinux, AppArmor, or a similar security framework on the host system, as these tools add an additional layer of protection against certain exploits. Lastly, user, group, and filesystem permissions should be properly configured on the host to limit access to the Kubernetes installation to authorized users only.

### **Keep your runtime secure and up-to-date**
No container runtime that works with Kubernetes is completely immune to security vulnerabilities, so it is not possible to guarantee that the runtime is completely secure. However, the risk can be minimized by regularly updating the runtime to the latest version.
### **Leverage logging and auditing to improve security**
Log data is essential for identifying potential security breaches and investigating past events. While Kubernetes offers facilities for generating log data, it does not provide any tools for auditing or analyzing that data for security purposes. Therefore, it is necessary to use third-party tools to make use of Kubernetes log data for security operations.

Sumo Logic makes it easy to aggregate and interpret Kubernetes logs, by installing the Sumo Logic Kubernetes App, teams can detect unusual activity on Kubernetes nodes and networks, and gain important visibility into their Kubernetes environments. With Sumo Logic, you can build end-to-end observability in Kubernetes:

1. Setup and Collection - The entire collection process can be set up with a single Helm chart, Fluentbit, Fluentd, Prometheus, and Falco are deployed throughout the cluster to collect log, metric, event, and security data.
2. Enrichment - Once collected, the data flows into a centralized Fluentd pipeline for metadata enrichment. Data is enriched- tagged- with the details about where it originated in the cluster; the service, deployment, namespace, node, pod, container, and their labels.
3. Sumo Logic - Finally, the data is sent to Sumo Logic via HTTP for storage, access, and most importantly analytics.

Labels - When you create objects in Kubernetes, you can assign custom key-value pairs to each of those objects, called labels. These labels can help you organize and track additional information about each object. For example, you might have a label that represents the application name, the environment the pod is running in or perhaps what team owns this resource. These labels are entirely flexible and can be defined as you need them. Our FluentD plugin ensures that those labels are captured along with the logs, giving continuity between the resources you have created and the log files they are producing.

## Metadata enrichment

Having unified metadata enrichment is essential for gaining a better understanding of the data in your cluster and the components' hierarchy. Standalone Prometheus or Fluentd deployments provide some context about the data - node, container, and pod level information - but not useful insight into the service, deployment, or namespace. Sumo Logic uses Open Telemetry to unify data collection.

By using Open Telemetry, Sumo Logic eventually unifies data collection with a single agent for logs, metrics, and traces. This eliminates vendor lock-in (such as with using Jaeger FluentBit, Dynatrace, New Relic, etc). Centralizing the collection method allows Sumo Logic’s solution to correlate and discover causality across your Kubernetes infrastructure.

A key aspect of Kubernetes monitoring is having consistent metadata tagging across logs, metrics, traces, and events, without which it would be impossible to correlate data when troubleshooting. You can use this metadata when searching through your logs and metrics and use them together to have a unified experience when navigating your machine data.

The namespace overview provides quick visibility into pods experiencing issues, or in this case, in a CrashLoopBackOff state. As many of you may already know from previously troubleshooting this common error, it is most often found due to over-utilized resources and memory usage. Correlating signals in order to find causality in this case is much simpler with the use of Open Telemetry via a single agent.


![](images/image36.png)


The namespace overview provides immediate insight into pods that are facing problems or are in a CrashLoopBackOff state.

