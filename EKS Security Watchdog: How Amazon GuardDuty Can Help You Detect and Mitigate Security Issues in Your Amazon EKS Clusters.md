# EKS Security Watchdog: How Amazon GuardDuty Can Help You Detect and Mitigate Security Issues in Your Amazon EKS Clusters


Starting from January 2022, Amazon GuardDuty began monitoring Amazon Elastic Kubernetes Service (Amazon EKS) clusters on a continuous basis in order to detect any malicious or suspicious behavior that could pose a threat to container workloads. Amazon GuardDuty for EKS Protection examines activity in the control plane by analyzing Kubernetes audit logs from both new and existing Amazon EKS clusters in your accounts. GuardDuty is integrated with Amazon EKS, which means it can access Kubernetes audit logs directly without the need for you to enable or store these logs. Once a threat is identified, GuardDuty generates a security finding that includes container details such as the pod ID, container image ID, and associated tags.


![](images/image13.png)



GuardDuty for EKS Protection includes 27 new types of findings that can help detect threats related to user and application activity captured in Kubernetes audit logs. This feature will be automatically enabled for all new and existing GuardDuty accounts, and will not require any additional configuration. AWS accounts will receive a 30-day free trial in each region to evaluate this new capability. During the free trial period, you can view your estimated EKS Protection spend in the GuardDuty console Usage page. You can also choose to suspend EKS Protection at any time by going to the Kubernetes Protection page in the GuardDuty console.
# **CredentialAccess:Kubernetes/SuccessfulAnonymousAccess**
The system has successfully performed an API task using the system:anonymous user, which does not require authentication. The use of this API is often linked to attempts by adversaries to gather passwords, user names, and access keys for a Kubernetes cluster. This suggests that anonymous or unauthenticated access is permitted for the specific API task reported and possibly others. If this is unexpected, it could be the result of a configuration error or a security breach involving the user's credentials.

# **DefenseEvasion:Kubernetes/SuccessfulAnonymousAccess**
This indicates that the system:anonymous user was able to successfully carry out an API operation without needing any authentication. API calls made using system:anonymous are not verified. The API is often utilized in attempts to conceal an attacker's actions in order to evade detection. This suggests that anonymous or unauthenticated access is allowed for the API action specified in the report and could be allowed for others as well. If this is not what was intended, it could mean there is a configuration error or that your credentials have been compromised.

# **Discovery:Kubernetes/SuccessfulAnonymousAccess**

The discovery shows that the system:anonymous user was able to perform an API operation without any authentication. This is concerning because the system:anonymous user can be used by anyone, allowing access to the API action reported in the finding, and possibly other actions as well. If this is not what was intended, it could be a result of a configuration error or a compromise of your credentials.
# **Impact:Kubernetes/SuccessfulAnonymousAccess**
The finding indicates that an API function was executed successfully by the system:anonymous user, indicating that the API requests made by system:anonymous are not verified. Additionally, the finding notes that the observed API is typically associated with the harm stage of a cyber attack. Therefore, if anonymous or unverified access is allowed on the API action reported in the finding, it may also be allowed on other actions.
# **Persistence:Kubernetes/SuccessfulAnonymousAccess**

The system:anonymous user successfully used an API operation. This means that the API calls made by system:anonymous are not verified. This activity is often seen when an attacker has gained access to your cluster and is trying to keep it. This shows that anonymous or unverified access is allowed on the API action mentioned in the finding and may be allowed on other actions. If this is not intended, it could be a mistake in configuration or a compromise of your credentials.


# **Policy:Kubernetes/AnonymousAccessGranted**
A user has made a ClusterRoleBinding or RoleBinding that grants access to some API operations for the system:anonymous user. This might be an error, or it might indicate that someone's credentials have been stolen.
# **Execution:Kubernetes/ExecInKubeSystemPod**
A command was run in a pod in the kube-system namespace using the Kubernetes exec API. The kube-system namespace is a default namespace used for system-level components, it is unusual to run commands inside pods or containers in the kube-system namespace, which may suggest suspicious activity.
# **Persistence:Kubernetes/ContainerWithSensitiveMount**
A container was started with settings that included a sensitive host path with write permission in the volumeMounts section. This allows sensitive host path to be accessible and modifiable from inside the container. This method is commonly used by attackers to gain access to the host's file system.
# **Policy:Kubernetes/AdminAccessToDefaultServiceAccount**
Kubernetes automatically creates a default service account for all namespaces in the cluster, and assigns the default service account as an identity to pods that do not have an explicit association with another service account. If the default service account has admin rights, it may cause pods to be launched unintentionally with admin rights, which may suggest a configuration error or a compromise of your credentials.
# **Policy:Kubernetes/ExposedDashboard**
The discovery shows that the Kubernetes dashboard for your cluster can be accessed from the internet through a Load Balancer service. This allows anyone to take advantage of any vulnerabilities in authentication and access control that may exist.
# **Policy:Kubernetes/KubeflowDashboardExposed**
The discovery reveals that the dashboard for your Kubeflow cluster has been made accessible to the public through a Load Balancer service. This makes the management interface of your Kubeflow environment available to anyone on the Internet, and could potentially allow malicious actors to take advantage of any weaknesses in the authentication and access control systems.
# **PrivilegeEscalation:Kubernetes/PrivilegedContainer**
This indicates that a container with administrative privileges was initiated on your Kubernetes cluster using an image that has never been employed to launch containers with elevated permissions before. This could be a technique used by a attacker to gain control and then damage the host.
# **CredentialAccess:Kubernetes/MaliciousIPCaller**
This message informs you that an attempt has been made to enter your Kubernetes cluster using a recognized harmful IP address. This specific IP address has a history of attempting to gather passwords, username, and keys for access.
# **CredentialAccess:Kubernetes/MaliciousIPCaller.Custom**
This discovery indicates that an attempt was made to enter your Kubernetes cluster using an IP address that is on a threat list that you have uploaded. The specific threat list related to this discovery can be found in the Additional Information section of the finding's details. The API that was utilized is commonly linked with obtaining credentials, in which someone is attempting to obtain passwords, usernames, and access keys for your Kubernetes cluster.
# **CredentialAccess:Kubernetes/TorIPCaller**

This discovery informs you that an API was invoked from a Tor exit node. This API is frequently used for obtaining passwords, usernames, and access keys for various environments. Tor is a software that enables anonymous communication by encrypting communications and transmitting them via various relays to different network nodes. The final Tor node is referred to as the exit node. This finding could suggest that someone without authorization accessed your Kubernetes cluster resources in an attempt to conceal their identity.
# **DefenseEvasion:Kubernetes/MaliciousIPCaller**

This implies that someone utilized an API in a manner that implies they were attempting to avoid detection. This is a common tactic used by adversaries who want to conceal their actions.
# **DefenseEvasion:Kubernetes/MaliciousIPCaller.Custom**
This discovery alerts you that an API operation was executed from an IP address that is included in a threat list that you have uploaded. The specific threat list related to this discovery can be found in the Additional Information section of the finding's details. The API in question is frequently linked to defense evasion techniques, where an adversary is attempting to conceal their actions in order to avoid detection.
# **DefenseEvasion:Kubernetes/TorIPCaller**
An API was executed from a Tor exit node IP address, which is frequently connected to defense evasion strategies where an adversary is attempting to conceal their actions to avoid detection. Tor is a software that allows anonymous communication by encrypting and randomly directing communications through relays among a series of network nodes. The final Tor node is referred to as the exit node. This can suggest unauthorized access to your Kubernetes cluster with the purpose of hiding the adversary's true identity.
# **Discovery:Kubernetes/MaliciousIPCaller**
A recent API operation on your Kubernetes cluster was executed from an IP address linked to recognized harmful activity. This type of operation is commonly used during the reconnaissance stage of an attack, when an attacker is collecting information to determine if your Kubernetes cluster is susceptible to a more comprehensive attack.
# **Discovery:Kubernetes/MaliciousIPCaller.Custom**
A request was made to an API from a known malicious IP address. This IP address is listed as a threat in relation to this incident. The API in question is often used during the reconnaissance phase of an attack, in which an attacker collects information to assess the vulnerability of a Kubernetes cluster to further attacks.
# **Discovery:Kubernetes/TorIPCaller**

A request was made to an API from an IP address associated with a Tor exit node. This is often used during the reconnaissance phase of an attack, when an attacker is gathering information to assess the vulnerability of a Kubernetes cluster to further attacks. This suggests that someone has gained unauthorized access to the cluster, likely with the goal of hiding their identity.
# **Impact:Kubernetes/MaliciousIPCaller**

A request was made to an API from an IP address with a history of malicious activity, this is likely an attempt to tamper with, disrupt or damage data within the AWS environment.
# **Impact:Kubernetes/MaliciousIPCaller.Custom**

An attempt was made to access your AWS environment from an IP address that is identified as a threat. The threat list can be found in the Additional Information section of the finding's details. The API that was accessed is commonly associated with tactics that aim to manipulate, disrupt, or destroy data within the AWS environment
# **Impact:Kubernetes/TorIPCaller**

The discovery indicates that an API was called from an IP address connected to a Tor exit node, which is frequently used in tactics where a attacker attempts to alter, disrupt, or destroy data within an AWS environment.
# **Persistence:Kubernetes/MaliciousIPCaller**

It implies that an individual has entered your Kubernetes cluster from an IP address that is related to recognized harmful actions, and they are probably attempting to preserve that access.
# **Persistence:Kubernetes/MaliciousIPCaller.Custom**

The finding notifies you that an API function was called from an IP address that is present on a list of potential threats. The list of potential threats is linked to this discovery and can be found in the Additional Information section of a discovery's details. This API function is typically connected with persistence strategies where an attacker has obtained access to your Kubernetes cluster and is trying to maintain that access.
# **Persistence:Kubernetes/TorIPCaller**

The discovery indicates that an individual has accessed your AWS resources through the Tor network to hide their identity. This may imply that the person is attempting to preserve access to your Kubernetes cluster and is utilizing strategies linked to maintaining access to a system.
