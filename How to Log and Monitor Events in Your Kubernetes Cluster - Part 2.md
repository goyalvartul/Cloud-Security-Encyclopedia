# How to Log and Monitor Events in Your Kubernetes Cluster - Part 2

## What are the different categories of events in Kubernetes?
Kubernetes events are divided into three main categories: informational, warning, and error. 

**Informational** events include when pods are scheduled, images pulled, a node is healthy, a deployment is updated, a replica set is called, and a container is killed. 

**Warning** events include when a pod has errors and persistent volumes are not yet bound. 

**Error** events include when a node is down, a persistent volume is not found, and a load balancer cannot be created in the cloud provider.


Events can be directly published by utilizing the REST API, API client or event recorder.


### **The most important Kubernetes events**
**Kubernetes has various types of events that are important to be aware of, some of which include:**

- **CrashLoopBackOff- Occurs when a pod repeatedly starts, crashes and restarts.**
- **ImagePullBackOff- Occurs when a node is unable to fetch an image.**
- **Evicted events - Happens when a node determines that a pod needs to be terminated or evicted to free up resources like CPU or memory. In such cases, Kubernetes will attempt to reschedule the pod on another node.**
- **FailedMount/FailedAttachVolume - When pods require persistent volumes or storage and this event occurs, it prevents the pod from starting if the storage is not accessible.**
- **FailedSchedulingEvents - Occurs when the scheduler is unable to find a node to run the pods.**
- **NodeNotReady- When a node can't run a pod due to an underlying issue.**
- **Rebooted**
- **HostPort conflict**



There are a variety of solutions that can be used to retrieve Kubernetes events. Some popular options include:

- Eventrouter
- Kubewatch 
- Sloop 
- Kubernetes event exporter
- Kspan.

**Eventrouter** is an active watcher of event resources in the Kubernetes system that pushes them to a user-specified sink for long-term behavioral analysis of workloads running on a Kubernetes cluster.

**Kubewatch** is a tool that tracks every resource change in a Kubernetes cluster and supports notifications through various platforms such as Slack, Hipchat, and Webhook.

**Sloop** is a tool that records the history of events and resource state changes in a Kubernetes cluster and provides visualizations to aid in debugging past events.

**The Kubernetes event exporter** is a simple but powerful solution that exports often missed Kubernetes events to various outputs for observability or alerting purposes.

**Kspan** is a project by Weaveworks that turns Kubernetes events into OpenTelemetry Spans, connecting them by causality and grouping them into traces. It then sends the generated traces to the OpenTelemetry collector.

