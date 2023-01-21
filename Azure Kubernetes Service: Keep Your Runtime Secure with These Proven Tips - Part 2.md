# Azure Kubernetes Service: Secure with These Proven Tips - Part 2


### **Falco for threat detection in AKS**

Using open source Falco for runtime threat detection allows you to identify and stop suspicious activity and anomalies in your container environment. Here are some examples:

Terminal shell in a container:

Falco detects when command-line Interface execution (terminal shell) takes place in a running container, in violation of a configured policy. This could be an indication of an attacker attempting to manipulate the system, download malware, or initiate other malicious activity. Below is an example of a Falco rule that detects this threat, which can help you meet compliance, auditing, and intrusion detection requirements.



    *- rule: Terminal shell in container*
    `  `*desc: >*
    `    `*A shell was used as the entrypoint/exec point into a container with an*
    `    `*attached terminal.*
    `  `*condition: >*
    `    `*spawned\_process and container*
    `    `*and shell\_procs and proc.tty != 0*
    `    `*and container\_entrypoint*
    `  `*output: >*
    `    `*A shell was spawned in a container with an attached terminal* 
    `    `*(user=%user.name %container.info shell=%proc.name parent=%proc.pname*
    `    `*cmdline=%proc.cmdline terminal=%proc.tty container\_id=%container.id* 
    `    `*image=%container.image.repository)*
    `  `*priority: NOTICE*
    `  `*tags: [container, shell, mitre\_execution]*

Unusual outbound network connection attempt:

If a typical system binary like ls or ps initiates an outbound TCP connection, it is a sign of an issue. A likely explanation is that the host has been infected with a rootkit. Detecting a rootkit installation when it happens (using the types of rules previously mentioned) is ideal. However, it is still essential to have multiple layers of defense and detect behaviors that can occur after an attack has begun.

In this case, here is a Falco rule that helps identify suspicious connections in AKS environments:



    \- rule: Unexpected outbound connection destination
    `  `desc: >
    `    `Detect any outbound connection to a destination outside of an allowed set
    `    `of ips, networks, or domain names
    `  `condition: >
    `    `consider\_all\_outbound\_conns and outbound and not
    `    `((fd.sip in (allowed\_outbound\_destination\_ipaddrs)) or
    `     `(fd.snet in (allowed\_outbound\_destination\_networks)) or
    `     `(fd.sip.name in (allowed\_outbound\_destination\_domains)))
    `  `output: >
    `    `Disallowed outbound connection destination 
    `    `(command=%proc.cmdline connection=%fd.name user=%user.name
    `    `container\_id=%container.id image=%container.image.repository)
    `  `priority: NOTICE
    `  `tags: [network]


MITRE ATT&CK framework detections:

Falco detects system events that appear abnormal based on the adversary tactics and techniques as outlined in the MITRE ATT&CK framework. From this information, activities identified as a threat or anomalous can be handled by isolating the affected pods and containers. The example below is for the detection of privilege escalation in AKS:


    \- rule: Launch Privileged Container
    `  `desc: >
    `    `Detect the initial process started in a privileged container. Exceptions are
    `    `made for known trusted images.
    `  `condition: >
    `    `container\_started and container
    `    `and container.privileged=true
    `    `and not falco\_privileged\_containers
    `    `and not user\_privileged\_containers
    `  `output: >
    `    `Privileged container started 
    `    `(user=%user.name command=%proc.cmdline %container.info 
    `    `image=%container.image.repository:%container.image.tag)
    `  `priority: INFO
    `  `tags: [container, cis, mitre\_privilege\_escalation, mitre\_lateral\_movement]


## How Sysdig Secure enhances runtime security in AKS:
Sysdig Secure is built on the open-source Falco detection engine. It expands upon what Falco offers to provide complete security throughout the container and Kubernetes lifecycle. Additionally, Sysdig Secure includes a user-friendly interface to make it easier to operate at a cloud-scale.

Ready-made runtime security policies for your container, Kubernetes, and cloud environments are available in Sysdig Secure. These policies are based on Falco rules and can be easily enabled or disabled within the Sysdig Secure user interface.

![](images/image14.png)

Each policy can be tailored to apply to specific areas of your deployments, take certain actions, and specify where notifications should be sent.

In the interface, you can view the Falco syntax to understand the structure of the Falco rule that the policy is based on. Additionally, Sysdig Secure provides an easy way to add more detections to an existing policy by choosing from the available rules library.



![](images/image15.png)


Monitoring and addressing security incidents on AKS:

With Sysdig Secure, DevOps and security operations teams can view a summary of events that have been triggered based on activity throughout the environment. (These events can also be sent to your preferred SIEM). You can delve deeper into policy violations to gain more context and access detailed records to assist with forensics and incident response.

For instance, an attempt to read sensitive files (e.g. files containing user, password, or authentication information) will trigger an alert event. You will then be able to identify where the problem is occurring, see what was accessed, understand the affected containers or services, and be able to further investigate the issue with all the necessary context.


![](images/image16.png)


To assist in managing and automating security for your AKS environment, Sysdig Secure also offers:

- Automated response actions such as killing, stopping or pausing containers for remediation
- Automated image profiling, a customizable policy editor, and centralized management capabilities
- Image scanning for CI/CD pipelines and registries to detect and prevent vulnerabilities in development and production
- Detailed system call captures that aid in forensics, incident response, and auditing
- Tools and visualizations that make creating Kubernetes network security policies easier
- Policies and checks aligned with compliance standards to ease compliance validation.


