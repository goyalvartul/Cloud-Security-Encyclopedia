# Kuber-locking Down Your Cluster: A Guide to Securing Your Kubernetes Environment - Part 1


As more and more companies, from small businesses to enterprises, adopt containerization and container orchestration, the need to protect any critical or sensitive infrastructure that runs container workloads has become more important.

As Kubernetes is the most widely-used container and container orchestration tool, it's important to discuss the security best practices that organizations should adopt to secure their Kubernetes clusters.


## 1. Upgrade Kubernetes to the latest version

One of the simplest yet overlooked ways to improve security is to regularly update Kubernetes environments. This includes utilizing the latest security updates and bug fixes, as well as testing the newest stable version in a non-production environment before deploying it to the production cluster.

## 2. Secure Kubernetes API server authentication

Kubernetes APIs serve as the main entry point to a Kubernetes cluster and can be accessed by administrators or service accounts through the command-line tool kubectl, REST API calls, or other client SDKs. The Kubernetes API server, also known as kube-apiserver, serves as the foundation of a cluster by providing access and ensuring proper functioning. 

As a best practice, all API calls within the cluster should use Transport Layer Security (TLS) encryption. Additionally, it is important to implement an API authentication mechanism that meets the access requirements of the cluster. Common authentication methods include certificates or bearer tokens. For large-scale, enterprise-level clusters, it is recommended to integrate with third-party OpenID Connect providers or Lightweight Directory Access Protocol servers to group users and control access. For more information on authenticating users and authentication strategies, refer to the official Kubernetes documentation.

## 3. Enable role-based access control authorization

Role-based access control (RBAC) is an access control mechanism that allows users and applications to perform specific actions based on the least-privilege model and enforces required permissions only. While it may take additional time to set up, it is essential for securing large-scale Kubernetes clusters running production workloads.

The following are some Kubernetes RBAC best practices that administrators should follow:

- To enforce RBAC as a standard configuration for cluster security, enable RBAC in the API server by passing the –authorization-mode=RBAC parameter.
- Use dedicated service accounts per application and avoid using the default service accounts created by Kubernetes. This enables administrators to enforce RBAC on a per-application basis and provides better control over granular access to each application resource.

Reduce optional API server flags to minimize the attack surface area on the API server. Each flag enables a certain aspect of cluster management and can expose the API server. Avoid using the following optional flags:

-anonymous-auth;
-insecure-bind-address; and
- -insecure-port.
- For an RBAC system to be effective, enforce the least privileges. By following the principle of least privilege and assigning only the necessary permissions to a user or application, everyone can perform their job effectively. Do not grant any additional privileges and avoid wildcard verbs ["\*"] or blanket access.
- Continuously update and adjust RBAC policies to avoid becoming outdated. Remove any permissions that are no longer required. While this may be tedious, it is essential for securing production workloads.

## 4. Control access to the kubelet

The kubelet is an agent that runs on each node of the cluster, interacting with users through a set of APIs that control the pods running on the nodes and performs specific operations. However, unauthorized access to the kubelet can give attackers access to the APIs and compromise node or cluster security. To reduce this attack surface and prevent unauthorized access to the APIs through the kubelet, you should follow these steps:

1. Disable anonymous access by setting the –anonymous-auth flag to false before starting the kubelet: --anonymous-auth=false.
1. Start the kube-apiserver --kubelet-client-certificate and --kubelet-client-key flags. This will ensure the API server authenticates to the kubelet and stops anonymous calls.
1. The kubelet provides a read-only API, which admins can access without authentication. This could expose potentially sensitive information about the cluster, so admins should close the read-only ports using the following command: --read-only-port=0.



