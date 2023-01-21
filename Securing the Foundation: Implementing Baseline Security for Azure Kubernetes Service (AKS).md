# Securing the Foundation: Implementing Baseline Security for Azure Kubernetes Service (AKS)

The Azure Security Benchmark version 1.0 offers suggestions on how to secure your Azure cloud solutions, and this guidance is also applied to Azure Kubernetes. The Azure Security Benchmark organizes the content by security controls, and the recommendations are relevant to Azure Kubernetes.

You can keep track of this security standard and its suggestions by using Microsoft Defender for Cloud. The Azure Policy definitions can be found in the Regulatory Compliance part of the Microsoft Defender for Cloud control panel.

The Azure Security Benchmark provides guidelines for securing cloud solutions on Azure, and this security baseline applies those guidelines to Azure Kubernetes. To monitor compliance with these guidelines, you can use Microsoft Defender for Cloud, which includes Azure Policy definitions in its Regulatory Compliance section. Some recommendations may require a paid plan to implement.

## **Network Security**

### **1.1: Protect Azure resources within virtual networks**
When you create an AKS cluster, it automatically creates a network security group and route table to handle traffic flow for services created with load balancers, port mappings, or ingress routes. These groups are associated with the virtual NICs on customer nodes and the subnet on the virtual network, respectively. This is done to ensure appropriate traffic flow.

Use AKS network policies to control network traffic by creating rules for incoming and outgoing traffic between Linux pods in a cluster, based on chosen namespaces and label selectors. To use network policies, the Azure CNI plugin and virtual network and subnets must be configured and it can only be enabled when creating a new AKS cluster, it cannot be added to an existing one.

You can create a private AKS cluster that ensures all network traffic between the AKS API server and node pools stays within the private network. The control plane, or API server, is located in an AKS-managed Azure subscription and uses internal IP addresses, while the customer's cluster or node pool is in their own subscription. The server and the cluster or node pool communicate with each other using the Azure Private Link service in the API server virtual network and a private endpoint that is available in the subnet of the customer's AKS cluster. Another option is to use a public endpoint for the AKS API server but limit access using the AKS API Server's Authorized IP Ranges feature.

### **1.2: Monitor and log the configuration and traffic of virtual networks, subnets, and NICs**
Utilize Microsoft Defender for Cloud and implement its network security suggestions to safeguard the network resources utilized by your AKS clusters. Turn on network security group flow logs and transfer the logs to an Azure Storage account for monitoring. Additionally, you can send the flow logs to a Log Analytics workspace and use Traffic Analytics to gain insight into traffic patterns in your Azure cloud, identify high-activity areas and security risks, comprehend traffic flow patterns, and pinpoint network misconfigurations.


### **1.3: Protect critical web applications**
The guidance suggests using Azure Application Gateway and its built-in Web Application Firewall (WAF) in front of an AKS cluster to provide an extra layer of security by filtering incoming traffic to web applications. It also suggests using an API gateway for managing access, throttling, caching, transforming, and monitoring for APIs used in the AKS environment. This can help decouple clients from microservices and reduce the complexity of managing cross-cutting concerns.


### **1.4: Deny communications with known malicious IP addresses**
To enhance security for your Azure Kubernetes Service (AKS) clusters, use Microsoft Defender for Cloud and implement its network protection recommendations. Use network security group flow logs and send them to an Azure Storage account for auditing or to a Log Analytics workspace for traffic analysis. Additionally, use an Azure Application Gateway with Web Application Firewall (WAF) to filter incoming traffic and an API gateway for authentication, authorization, and monitoring of APIs. Enable Microsoft DDoS Standard protection on the virtual networks where AKS components are deployed and use Kubernetes network policies to control the flow of traffic between pods in AKS. Define rules that limit pod communication based on labels, namespaces, or traffic ports. Network policies can be automatically applied as pods are created in the AKS cluster.


### **1.5: Record network packets**
Use Network Watcher's packet capture feature as needed to investigate unusual activity. Network Watcher is automatically enabled in your virtual network's region when creating or updating a virtual network in your subscription. It can also be set up using PowerShell, Azure CLI, REST API, or Azure Resource Manager Client.


### **1.6: Deploy network-based intrusion detection/intrusion prevention systems (IDS/IPS)**
To secure your AKS cluster, use an Azure Application Gateway that has a Web Application Firewall (WAF) enabled. This can be set up in "detection mode" to log alerts and threats or "prevention mode" to block detected intrusions and attacks. This is a good option if you do not need intrusion detection and/or prevention based on payload inspection or behavior analytics.

### **1.8: Minimize complexity and administrative overhead of network security rules**
Use virtual network service tags to set up network access controls on the network security groups associated with AKS clusters. Instead of using specific IP addresses, you can use service tags to define rules for allowing or blocking traffic to the corresponding service by specifying the service tag name. Microsoft manages the address prefixes included in the service tag and updates them automatically when addresses change. Additionally, apply an Azure tag to the node pools in your AKS cluster. This is different from virtual network service tags and is applied to each node within the pool and remains through upgrades.

### **1.9: Maintain standard security configurations for network devices**

To ensure the security of your Azure Kubernetes Service (AKS) clusters, it is important to define and implement standard security configurations with Azure Policy for the network resources associated with your AKS clusters. To do this, you can use Azure Policy aliases in the "Microsoft.ContainerService" and "Microsoft.Network" namespaces to create custom policies to audit or enforce the network configuration of your AKS clusters. Additionally, there are built-in policy definitions related to AKS that can be used, such as:

- Defining authorized IP ranges on Kubernetes Services
- Enforcing HTTPS ingress in the Kubernetes cluster
- Ensuring that services listen only on allowed ports in the Kubernetes cluster

