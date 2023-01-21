# Azure Kubernetes Service: Keep Your Runtime Secure with These Proven Tips - Part 1

To ensure the security of your Azure Kubernetes Service (AKS) environment, it's important to implement controls that can detect any abnormal or malicious activity across your applications, infrastructure, and cloud environment. This includes safeguarding against potential runtime threats such as:

- Vulnerabilities that have yet to be patched or newly discovered exploits
- Configuration settings that are not secure
- Credentials that have been leaked or are weak
- Any unauthorized activity

Even with the use of tools like container image vulnerability scanning, Kubernetes pod security policies, and Kubernetes network policies within AKS, it's important to have additional measures in place to confirm that the security measures you have implemented are effective. Additionally, having a final line of defense can aid in identifying any zero-day vulnerabilities or unexpected activity.




## **Why runtime security with AKS?**

"Container security" is often associated with image scanning and vulnerability management, as it is believed that by scanning images for vulnerabilities, misconfigurations, and compliance violations early in the CI/CD pipeline, one can prevent threats in the container and Kubernetes environment. However, additional security measures are required. Therefore, runtime security, zero-trust networking, audit, forensics, and incident response should also be incorporated in your secure DevOps workflow.

Several security threats manifest during runtime, which include:

- Zero-day vulnerabilities – issues previously unknown to a software creator
- Privilege escalation attempts
- Bugs that cause erratic behavior or resource leaking

This article will discuss how to implement runtime security detection using an open-source-based approach with the Falco project, and how Sysdig Secure builds on Falco to detect and alert on runtime threats at scale in AKS environments.



## **Falco: Open-source Kubernetes runtime detection**

Falco, a cloud-native runtime security project created by Sysdig in 2016, is a widely used Kubernetes threat detection engine and the first runtime security project to become an incubation-level project at the Cloud Native Computing Foundation (CNCF). Its purpose is to detect any abnormal application behavior and alert on potential threats during runtime.

Falco operates by analyzing kernel system calls to provide in-depth insight into container and environment activity. Additionally, it incorporates other sources of data such as Kubernetes API audit events. Furthermore, Falco adds relevant Kubernetes application context to its findings, which helps DevOps, security and cloud teams understand the details of who did what and where.



### **Why Falco for runtime security in Azure Kubernetes Service?**

One limitation of commonly used vulnerability and malware detection methods is that they rely on pre-existing knowledge of potential threats. For instance, signature-based approaches require listing each possible exploit, vulnerability, or attack in some form (e.g., malware signatures). This means that the detection tool is constantly playing catch-up with a never-ending stream of new threats.

Behavioral approaches, on the other hand, focus on what is happening on a system, detecting the actions of a user or attacker once they have access. Falco detects any abnormal or malicious application and environment behavior using security rules. These rules can be obtained from the community or can be written by the user using Falco's flexible rules engine. Falco can then be integrated into security detection and response workflows to identify and alert on runtime threats.

A Falco rules file is a YAML file containing three types of elements:


**Rules** specify the circumstances under which an alert should be generated. Each rule is accompanied by a descriptive output message that is sent along with the alert.

**Macros** are reusable snippets of rule conditions that can be used within rules and other macros.

**Lists** are groups of items that can be included in rules, macros, or other lists. Unlike rules and macros, lists cannot be used as filtering expressions.


