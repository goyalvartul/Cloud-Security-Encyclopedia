# How to Log and Monitor Events in Your Kubernetes Cluster - Part 1

Kubernetes can be used to gather metrics, logs, and events for monitoring and troubleshooting purposes.

Events are a valuable source of information for understanding what is happening in your services, and there are various tools that can be used to make the most of them.

In this post, we will delve into the topic of Kubernetes events, including:

- How the events mechanism works
- The structure of events in the Kubernetes API
- The types of events to pay attention to
- The solutions available for retrieving events.


## **Introduction to Kubernetes events**


Kubernetes generates a large amount of events related to the deployment, scheduling, and other activities of our workloads. These events provide valuable information for understanding what is happening in our cluster and can help to answer questions such as "Why was this particular pod killed or restarted?"

There are two main ways to view events in Kubernetes: 

- using the command "kubectl describe pod <podname>" or 
- "kubectl get events".


` `When troubleshooting issues in your application, it is important to first check the events and infrastructure operations. It can also be useful to keep events for a longer period of time for post-mortem analysis or understanding if a failure was caused by previous events.

There are various types of events in Kubernetes, as every Kubernetes object goes through multiple states before reaching its desired state.

The master node and worker node have several core components that allow Kubernetes to orchestrate the workload on our servers. The scheduler schedules pods on the nodes, the control manager detects state changes and reschedules pods in case of crashes, and etcd stores the status of various Kubernetes resources (but only for the last hour).

All of these core components can orchestrate our workload based on events, making events crucial for understanding a given situation.

For example, when deploying a pod, the scheduler attempts to identify the correct node to start the pod. While this is happening, the pod will be in a "pending" state. Once the correct node is identified, the pod will be in a "creating" state. To start the pod, the node must first pull the container's image from an external docker registry. The scheduler prefers scheduling pods on nodes that already have the image. Once the image is pulled, the pod will be in a "running" state.

If the pod crashes, the control manager will reschedule it. However, if the pod crashes multiple times with the same error, it will enter a "CrashLoopBackOff" state. If nodes are stuck in a "pending" state, it may indicate that there are no available resources or that it is impossible to find the correct node.

Pods usually have health probes or readiness probes that help Kubernetes determine the state or health of the pod, such as "/health" or "/ready". The Kubelet is responsible for checking these endpoints. It is also possible to define an init container with a specific image, which Kubernetes will start and run before the other containers. If the image specified in the deployment file is incorrect or there is a connectivity issue with the docker registry, the node will not be able to pull the image and the pod will not reach the "running" state. In this case, the event "ImagePullBackOff" will be displayed.


### **Events in the Kubernetes API**
All events can be accessed through the Kubernetes API, which can be done using kubectl. The "kubectl describe" command is often used to gather information such as the status, reason, and other details about an event. When using the API, you can retrieve the following information about an event:

- The message
- The reason
- The type
- The object involved in the event
- The number of occurrences of the event

The source of the event (such as kubelet or other)

- This is the same information that can be obtained when using kubectl to view events.



