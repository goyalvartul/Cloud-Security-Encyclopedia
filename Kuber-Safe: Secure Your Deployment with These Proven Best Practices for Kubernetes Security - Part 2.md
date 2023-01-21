# Kuber-Safe: Secure Your Deployment with These Proven Best Practices for Kubernetes Security - Part 2


**Define Resource Quota**

Running containers without resource constraints can put your system at risk of DoS or "noisy neighbor" scenarios. To prevent and minimize these risks, you should define resource quotas. By default, all resources in a Kubernetes cluster are created with unbounded CPU and memory requests/limits. You can create resource quota policies, which are attached to a Kubernetes namespace, to limit the CPU and memory a pod is allowed to consume.

The following is an example of a namespace resource quota definition that limits the number of pods in the namespace to 4, with CPU requests between 1 and 2 and memory requests between 1GB to 2GB.

compute-resources.yaml:


    apiVersion: v1  
    kind: ResourceQuota  
    metadata:  
    `  `name: compute-resources  
    spec:  
    `  `hard:  
    `    `pods: "4"  
    `    `requests.cpu: "1"  
    `    `requests.memory: 1Gi  
    `    `limits.cpu: "2"  
    `    `limits.memory: 2Gi

Assign a resource quota to namespace:

    kubectl create -f ./compute-resources.yaml --namespace=myspace


**Implement Network Segmentation**

Running different applications on the same Kubernetes cluster can create a risk of one compromised application attacking another neighboring application. Network segmentation is crucial to ensure that containers can communicate only with those they are supposed to.

One of the challenges in Kubernetes deployments is creating network segmentation between pods, services, and containers. This is challenging due to the dynamic nature of container network identities (IPs) and the fact that containers can communicate both within the same node or between nodes.

Users of Google Cloud Platform can benefit from automatic firewall rules that prevent cross-cluster communication. A similar implementation can be deployed on-premises using network firewalls or SDN solutions. The Kubernetes Network SIG is working on improving pod-to-pod communication policies. A new network policy API aims to address the need to create firewall rules around pods, limiting the network access that a container can have.

The following is an example of a network policy that controls the network for "backend" pods, only allowing inbound network access from "frontend" pods:

POST /apis/net.alpha.kubernetes.io/v1alpha1/namespaces/tenant-a/networkpolicys  

    {  
    `  `"kind": "NetworkPolicy",
    `  `"metadata": {
    `    `"name": "pol1"
    `  `},
    `  `"spec": {
    `    `"allowIncoming": {
    `      `"from": [{
    `        `"pods": { "segment": "frontend" }
    `      `}],
    `      `"toPorts": [{
    `        `"port": 80,
    `        `"protocol": "TCP"
    `      `}]
    `    `},
    `    `"podSelector": {
    `      `"segment": "backend"
    `    `}
    `  `}
    }


**Apply Security Context to Your Pods and Containers**

When designing your containers and pods, it is important to configure the security context for your pods, containers, and volumes. A security context is a property defined in the deployment yaml file, it controls the security parameters that will be assigned to the pod/container/volume. Some of the important parameters include:

|**Security Context Setting**|**Description**|
| :-: | :-: |
|SecurityContext->runAsNonRoot|Indicates that containers should run as non-root user|
|SecurityContext->Capabilities|Controls the Linux capabilities assigned to the container.|
|SecurityContext->readOnlyRootFilesystem|Controls whether a container will be able to write into the root filesystem.|
|PodSecurityContext->runAsNonRoot|Prevents running a container with 'root' user as part of the pod|


    The following is an example for pod definition with security context parameters:
    
    apiVersion: v1  
    kind: Pod  
    metadata:  
    `  `name: hello-world  
    spec:  
    `  `containers:
    `  `# specification of the pod’s containers  
    `  `# ...  
    `  `securityContext:  
    `    `readOnlyRootFilesystem: true  
    `    `runAsNonRoot: true


If you are running containers with elevated privileges (--privileged), it is recommended to use the "DenyEscalatingExec" admission control. This control denies exec and attach commands to pods that run with escalated privileges that allow host access. This includes pods that run as privileged, have access to the host IPC namespace, and have access to the host PID namespace. For more information on admission controls, refer to the Kubernetes documentation.

**Log Everything**

Kubernetes provides centralized logging for container activity. When a cluster is created, the standard output and standard error of each container can be collected using a Fluentd agent running on each node and sent to either Google Stackdriver Logging or Elasticsearch, which can then be viewed using Kibana.


