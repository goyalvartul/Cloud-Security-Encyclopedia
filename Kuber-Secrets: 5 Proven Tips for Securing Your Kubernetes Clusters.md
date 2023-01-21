# Kuber-Secrets: 5 Proven Tips for Securing Your Kubernetes Clusters

Despite being relatively young, the Kubernetes project has made a significant impact on the ability for organizations to effectively respond to user demands. It's important to remember that the project is still relatively new and as a community, we should continue to work towards maturity.

It is hard to believe that Kubernetes is still a relatively new project despite its significant impact on businesses and governments. As a community, we may not be fully mature yet. Recently, when I explained to a developer that deploying a container running as root and requiring port 80 poses a security risk, the developer's casual response was to suggest removing restrictions from the cluster. This is concerning and highlights a lack of understanding about the importance of security in Kubernetes.

It is common to encounter developers who may not fully understand the security risks associated with building containers that require privileged access. This can lead to an increased security threat for the organization. However, it is possible to address this issue and it does not have to be difficult.

### **Why is the prevalence of privileged containers an issue?**

A container that runs with privileged access, or as the root user, poses a significant security risk because it has the ability to access and potentially compromise the host and other applications. This type of container, if compromised, could mount all disks on the host and gain read-write access to the files on those volumes, potentially causing major issues for the organization.

Enterprise Kubernetes distributions like OpenShift help to reduce the risk of privileged containers by running them in a "restricted" mode as a default, and also by adding an additional layer of security using Security-Enhanced Linux (SELinux). In most cases, microservices do not require more permissions beyond this "restricted" mode. There are only a few instances where more permissions are necessary. This policy in Red Hat OpenShift specifically helps to protect users from container breakout vulnerabilities such as CVE-2019-5736: runc CONTAINER BREAKOUT.

**Creating more secure containers is not difficult.** 

There are many resources and base container images that can guide developers towards better security practices. These containers can easily run on local docker or podman instances and on less secure Kubernetes clusters. However, when it comes to deploying on enterprise-ready Kubernetes clusters, these practices may not be sufficient. To improve security, here are some tips to follow:

### Tip 1
To improve the security of containers, it is important to avoid running them as the root user. Instead, containers should be built to support non-root user IDs. This can be done by setting the permissions of copied files to a specific user ID, such as 1001, and then executing the container using that user ID. This can be done by using the USER directive in the container's build file.

![images/image17.png)

### Tip 2

Instead of using direct communication between containers, use services for inter-image communication. This will provide you with the added advantage of Kubernetes orchestration capabilities.

### Tip 3

Avoid using ports below 1024 - These ports are considered privileged and can pose a security risk. It is easy to change the ports used in a container image, so assign a non-privileged port, such as 8080, in the container image and map it to a privileged port, such as 80 or 443, for external access when running the container using podman or Kubernetes.

### Tip 4

Use base images that have been built following established security practices and from a reputable source. This will ensure the image has been constructed in a secure manner and that you can trust its origins. Utilizing a Red Hat developer account to access Red Hat UBI images is one option, as well as using application-specific images. Other options include using images based on Centos or Fedora.

### Tip 5

Consider using the source-to-image capabilities - S2I images allow you to focus on the code, and either the s2i-SDK or OpenShift will build the containers for you. This approach incorporates best practices for building containers, making it a great option for new Kubernetes developers. Security teams often prefer this approach as it allows for a higher level of standardization.


