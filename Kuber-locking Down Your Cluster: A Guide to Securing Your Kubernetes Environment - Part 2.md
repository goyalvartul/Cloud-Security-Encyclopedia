# Kuber-locking Down Your Cluster: A Guide to Securing Your Kubernetes Environment - Part 2



## 5. Harden node security

In order to strengthen the security of the nodes on which the pods are running, it is important to begin by implementing the following measures: establish clear configuration standards and guidelines, properly configure the host in accordance with security best practices, and regularly validate the cluster against the Center for Internet Security's benchmarks that are specific to the Kubernetes release in use. 

Additionally, it is crucial to make sure that all security protocols and measures are kept up to date and that any vulnerabilities or weaknesses are promptly identified and addressed. Furthermore, it is important to establish a robust monitoring system to detect any security breaches or anomalies in a timely manner and to have a well-defined incident response plan in place to respond to any security incidents that may occur.

**Admin access minimization.** 

One way to decrease the potential for attack on a Kubernetes system is to limit the amount of administrative access on the nodes. One way to do this is through the use of node isolation and restrictions, by running certain pods on specific nodes or groups of nodes. This allows for the pods to be run on nodes that have been configured with specific isolation and security settings. Another approach to controlling which nodes a pod can access is to add labels to the node objects, this way pods can be targeted to specific nodes. This can be done by using the below command:

` `"kubectl label nodes <node name> <label key>=<label value>" 

Once the label has been applied to the node, you can use a nodeSelector in the pod deployment in a YAML file to ensure that the pod is deployed to the designated node.



*apiVersion: v1*

*kind: Pod*

*metadata:*

`  `*name: nginx*

`  `*labels:*

`	`*env: staging*

*spec:*

`  `*containers:*

`  `*- name: nginx-staging*

`	`*image: nginx*

## 6. Set up namespaces and network policies

Using namespaces can help separate workloads that contain sensitive information from those that do not. While managing multiple namespaces may be more complex, it simplifies the process of applying security controls, such as network policies, to sensitive workloads. This allows for greater control over the flow of traffic to and from pods. By isolating sensitive workloads in specific namespaces, it can be easier to implement security measures to protect the sensitive data and prevent unauthorized access.

## 7. Enable audit logging

It is essential to enable audit logs for Kubernetes clusters and to constantly monitor them for any suspicious or malicious activity or API calls. Audit logs are an effective tool in detecting potential security issues in real-time. They can keep granular records of actions performed in the cluster, such as an attacker trying to brute force a password, which would generate authentication and authorization-related logs. If these logs are repetitive, it could indicate a security issue.

Audit logs are not enabled by default, so to activate them, use the Kubernetes audit policy. The audit policy allows administrators to select one of the four audit levels:

- None: No events that match this rule are logged
- Metadata: Request metadata such as requesting user, timestamp, resource, and verb are logged
- Request: Event metadata and request body but not response body are logged. This does not apply to non-resource requests.
- RequestResponse: Event metadata, requests, and response bodies are logged. This does not apply to non-resource requests.

To enable audit logs on Kubernetes clusters, follow these steps:

- Begin by logging into the master node via SSH
- Create an audit log policy file using YAML, and save it as a yaml file.
- Create a new directory on the master node to store the audit logs, for example, at /kube/auditlogs/.
- Configure the kube-apiserver to load the audit policy by editing the manifest file located at /etc/kubernetes/manifests/kube-apiserver.yaml, and add the -audit-policy-file flag to the policy YAML created in step 2. Or define -audit-log-path to direct audit logs to a specific file.
- Save and exit the file.

