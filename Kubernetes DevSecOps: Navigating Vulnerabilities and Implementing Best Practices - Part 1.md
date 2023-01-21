# Kubernetes DevSecOps: Navigating Vulnerabilities and Implementing Best Practices - Part 1

The use of containers and Kubernetes has significantly improved the process of deploying cloud-native applications for many teams. The ability to identify issues in the deployment pipeline and utilize machine intelligence to detect security threats has become more efficient since the introduction of the Cloud Native Computing Foundation's Open Telemetry project.

The use of a popular open-source container orchestration tool such as Kubernetes offers many advantages, but it also exposes new vulnerabilities in the CI/CD pipeline. The standardization of telemetry is essential in identifying and addressing common or known issues, however, it does not fully address every potential challenge.

One of the major challenges that arise with the use of containers and Kubernetes is maintaining continuous security. By increasing the number of layers and complexity in application environments, containers and Kubernetes create new possibilities for attackers and pose new risks for Kubernetes administrators to manage. Even though Kubernetes has some built-in security features, they are not sufficient enough to prevent all types of attacks on their own.

The following provides a summary of the essential security measures for Kubernetes, including the primary categories of security threats that can occur in a Kubernetes environment, the reasons why securing Kubernetes is more challenging than securing non-containerized applications, and recommended security practices that teams can implement to enhance the security of their Kubernetes containers.

## Kubernetes vulnerabilities
Securing Kubernetes can be difficult as it is a complex platform composed of various components, each of which poses its own set of security challenges and risks. The following list provides an overview of the major components of a Kubernetes environment and the typical security risks that are associated with them:

- **Containers**: Containers can include potentially harmful code that was included in the container image, or may be configured incorrectly, leaving them vulnerable to unauthorized access.
- **Host operating systems**: The operating systems running on the Kubernetes nodes can contain vulnerabilities or malicious code that can give attackers access to the Kubernetes cluster.
- **Container runtimes**: Kubernetes supports various container runtimes, all of which have the potential to contain vulnerabilities that allow attackers to take control of individual containers, escalate attacks to other containers, and even gain control of the entire Kubernetes environment.
- **Network layer**: Kubernetes uses internal networks for communication between nodes, pods and containers, and also typically exposes applications to public networks for Internet access. These network layers can provide attackers with a means of accessing the cluster or escalating attacks from one area to another.
- **API**: The Kubernetes API, which is crucial for communication and configuration, can contain vulnerabilities or misconfigurations that can be exploited by attackers.
- **Kubectl** (and other management tools): Kubectl, Dashboard, and other Kubernetes management tools may have vulnerabilities that can be abused on the Kubernetes cluster.

## Built-in Kubernetes security features

Kubernetes has built-in security features to address potential threats and minimize the impact of a breach. These include:

- Role-based access control (RBAC) allows administrators to define roles and cluster roles that determine which users have access to specific resources within a namespace or across the entire cluster. RBAC is considered a best practice for regulating access to resources.
- Pod security policies and network policies enable administrators to restrict the behavior of containers and pods. For example, pod security policies can be used to prevent containers from running as the root user, and network policies can limit communication between pods.
- Network encryption is achieved through the use of Transport Layer Security (TLS), which encrypts network traffic to protect against eavesdropping. This is a commonly used security best practice, and it is also used to secure HTTPS, email, and messaging platforms.

While Kubernetes has built-in security features, they do not provide complete protection against all types of attacks. Kubernetes primarily uses declaratively run environments, so it does not offer native protections against the following:

- **Malicious code or misconfigurations inside containers or container images -**  You would need to use a third-party container scanning tool to detect these.
- **Shadow IT deployments or changes that bypass the company's proper change management system and compliance-** This can create significant Kubernetes security challenges.
- **Security vulnerabilities on host operating systems -**  These need to be scanned for using other tools. Some Kubernetes distributions, like OpenShift, integrate SELinux or similar kernel-hardening frameworks to provide more security at the host level, but this is not a feature of Kubernetes itself.
- **Container runtime vulnerabilities -** Kubernetes has no way to detect or alert you if a vulnerability exists within your runtime or if an attacker is trying to exploit a vulnerability in the runtime.
- **Abuse of the Kubernetes API-**  Kubernetes does not detect or respond to API abuse beyond following any RBAC and security policy settings that you define.
- **Management tool vulnerabilities or misconfiguration -** . Kubernetes cannot guarantee that management tools like Kubectl and Helm chart deployments are free of security problems.