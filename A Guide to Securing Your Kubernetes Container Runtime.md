# A Guide to Securing Your Kubernetes Container Runtime


Although many administrators recognize the significance of runtime security and its importance, they tend to neglect or not allocate enough resources towards it in their Kubernetes security plans. 

The complexity of Kubernetes and the variety of native tools it provides for managing access rights and separating workloads can lead to a focus on securing the cluster itself rather than the runtime security of Kubernetes. 

This article aims to bridge that gap by discussing the crucial role of runtime security in Kubernetes and providing information on the tools available for protecting against runtime threats.

## **What Is Runtime Security in Kubernetes?**
Runtime security in Kubernetes involves safeguarding containers (or pods) from active threats once they are in operation. It is responsible for protecting workloads against potential risks that may arise after container deployment, such as:

- The emergence of malware within a container image.
- Attempts of privilege escalation where a container takes advantage of security flaws in the container runtime, Kubernetes, or the host OS to gain access to resources it should not have access to (e.g. storage volumes or external binaries).
- Deployment of unauthorized containers by an attacker who exploits a gap in an access control policy or a bug in Kubernetes.
- Unauthorized access to secrets or other sensitive information that a running container should not be able to read, but which it manages to access due to improper access control configurations or a security vulnerability that enables privilege escalation.

Ideally, if the security of application pipelines and clusters were implemented effectively, runtime threats would not occur as they would not be able to infiltrate live environments. Additionally, strict access controls would ensure proper isolation of workloads, preventing any harm from a compromised container to the cluster. However, it is impossible to guarantee that attackers will not find ways to sneak malware into container images, for example, by compromising source code repositories or build tools. Moreover, Kubernetes RBAC policies and security contexts cannot ensure complete isolation between workloads.

That is why runtime security is vital; it acts as a final layer of protection against threats that may bypass other security measures. While it is important to take all reasonable precautions to reduce security risks within the pipeline and cluster architecture, runtime security is also necessary to detect and control threats that manage to slip through.



## **Kubernetes Runtime Security Tools**
Kubernetes does not provide many options for runtime security tools. The only built-in feature it has is auditing, which allows to create logs that record resource requests to the Kubernetes API. However, Kubernetes does not provide any means to analyze audit logs or alert you to potentially suspicious activity.

Therefore, to protect the environment from runtime threats, external runtime security tools should be used instead of relying on Kubernetes. These tools can be divided into two main categories for Kubernetes: enforcement tools and auditing tools.

### **Runtime Security Enforcement Tools**

Enforcement tools provide the capability to establish policies that limit access rights and permissions of resources within a runtime environment. While these tools cannot prevent a threat from occurring, they can mitigate its impact by ensuring that, for example, malware that appears in one container is unable to access resources outside of the container.
### **Key enforcement tools for Kubernetes include:**

- **Seccomp**: It is a Linux kernel-level tool that can be utilized to force processes to run in a secure state. When Seccomp is used, the kernel prevents the process from making any system calls except for exit, sigreturn, and reading and writing to open files.
- **SELinux**: This is a kernel module that allows the definition of a wide range of access controls that the kernel enforces. SELinux can be used to establish granular rules on the resources a container can access and the types of actions it can perform.
- **AppArmor**: Similar to SELinux, it is a kernel module that enables the definition of a broad set of access control rules. It is very similar to SELinux, and the debate about which to use is similar to the one about vi vs. emacs - some people prefer one over the other, but both tools can achieve similar results.


To some degree, Kubernetes RBAC policies and security contexts can also be considered as providing enforcement control as they can be used to prevent a container from running in privileged mode or deny access to kernel-level resources.

However, the difference between RBAC policies and security contexts and tools such as SELinux and AppArmor is that the latter enforces access controls at the kernel level. This means that even if Kubernetes is compromised or has a security vulnerability, kernel-level enforcement tools can still minimize the impact of a security breach by preventing compromised containers from accessing external resources.

SELinux, AppArmor, and seccomp are also beneficial as they can be used to control access for any Linux workloads, not only specific to Kubernetes. They can be useful if managing an environment that includes a mix of containers and VMs, as the same tooling can be used to manage all workloads.

### **Auditing Tools for Runtime Security**

Auditing tools in Kubernetes assist in identifying and responding to threats by utilizing data collected from the cluster. Although Kubernetes has the capability to create audit logs, it does not have the function to analyze the logs. A tool such as Falco, an open-source threat detection engine, can be used to accomplish that. Falco allows to set rules that will generate alerts if the engine detects specific conditions based on data such as Kubernetes audit logs. 

For example, a Falco rule like:

\- **rule**: Example **rule** (nginx). This **is** the human **name** **for** the **rule**.

**desc**: Detect **any** listening socket outside our expected one.

condition: evt.**type** **in** (accept,**listen**) **and** (container.

image!=myregistry/nginx **or** proc.name!=nginx **or** k8s.ns.name!=”**load**-

balancer”)

output: This **is** **where** I **write** the alert message **and** I provide **some**

extra information (command=%proc.cmdline **connection**=%fd.name).

priority: WARNING


The rule states that if any listening sockets do not match the specified criteria, an alert will be generated. These criteria include:

- The image used must be myregistry/nginx.
- The process listening must be nginx.
- The namespace must be load-balancer.

This rule can help identify any containers in the environment that may be attempting to collect data or access unauthorized resources, as well as detect unauthorized containers.

Falco is widely used as an auditing tool for Kubernetes and other similar platforms, due to its open source nature and ability to work with any type of cloud-native environment. Additionally, the Cloud Native Security Hub repository offers preconfigured Falco rules for various workloads, making it easy to quickly set up Falco without the need to manually write multiple auditing rules.


## **Best Practices for Kubernetes Runtime Security**

Securing Kubernetes runtime environments involves more than just deploying enforcement and auditing tools. In addition to these tools, it is important to:

- Continuously scan images to prevent malware from entering the runtime environment.
- Scan Kubernetes policy files, deployments, and other resources to detect misconfigurations that could enable or exacerbate a runtime breach.
- Check external policy files or profiles that you create for SELinux, AppArmor, or other frameworks to prevent oversights that could weaken runtime defenses.
- Secure development and test environments as well as production, as catching runtime threats in dev/test can prevent them from reaching production and even in dev/test environment a runtime security issue could have major consequences.
- Develop a plan for incident remediation, don’t wait until a breach occurs, so that your team can handle different runtime security events in a timely and effective manner.

Runtime security is critical to protect against threats that have bypassed other defenses put in place earlier in the application development pipeline and Kubernetes architecture. By continuously auditing and enforcing segmentation of runtime resources, the potential impact of runtime threats can be minimized.

