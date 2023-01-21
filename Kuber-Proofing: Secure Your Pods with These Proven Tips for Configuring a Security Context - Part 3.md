# Kuber-Proofing: Secure Your Pods with These Proven Tips for Configuring a Security Context - Part 3


## Set capabilities for a Container
With Linux capabilities, you can grant certain privileges to a process without granting all the privileges of the root user. To add or remove Linux capabilities for a Container, include the capabilities field in the securityContext section of the Container manifest.

First, see what happens when you don't include a capabilities field. Here is configuration file that does not add or remove any Container capabilities:

pods/security/security-context-3.yaml 

    **apiVersion**: v1
    **kind**: Pod
    **metadata**:
    ` `**name**: security-context-demo-3
    **spec**:
    ` `**containers**:
    ` `- **name**: sec-ctx-3
    `   `**image**: gcr.io/google-samples/node-hello:1.0

Create the Pod:

    kubectl apply -f https://k8s.io/examples/pods/security/security-context-3.yaml

Verify that the Pod's Container is running:

    kubectl get pod security-context-demo-3

Get a shell into the running Container:

    kubectl exec -it security-context-demo-3 -- sh

In your shell, list the running processes:

    ps aux

The output shows the process IDs (PIDs) for the Container:

    USER  PID %CPU %MEM    VSZ   RSS TTY   STAT START   TIME COMMAND
    
    root    1  0.0  0.0   4336   796 ?     Ss   18:17   0:00 /bin/sh -c node server.js
    
    root    5  0.1  0.5 772124 22700 ?     Sl   18:17   0:00 node server.js

In your shell, view the status for process 1:

    cd /proc/1

    cat status

The output shows the capabilities bitmap for the process:

    CapPrm:	00000000a80425fb
    CapEff:	00000000a80425fb


Make a note of the capabilities bitmap, and then exit your shell:

    exit

Next, run a Container that is the same as the preceding container, except that it has additional capabilities set.

Here is the configuration file for a Pod that runs one Container. The configuration adds the CAP\_NET\_ADMIN and CAP\_SYS\_TIME capabilities:

pods/security/security-context-4.yaml 

    **apiVersion**: v1
    **kind**: Pod
    **metadata**:
    ` `**name**: security-context-demo-4
    **spec**:
    ` `**containers**:
    ` `- **name**: sec-ctx-4
    `   `**image**: gcr.io/google-samples/node-hello:1.0
    `   `**securityContext**:
    `     `**capabilities**:
    `       `**add**: ["NET\_ADMIN", "SYS\_TIME"]

Create the Pod:

    kubectl apply -f https://k8s.io/examples/pods/security/security-context-4.yaml

Get a shell into the running Container:

    kubectl exec -it security-context-demo-4 -- sh

In your shell, view the capabilities for process 1:

    cd /proc/1

    cat status

The output shows capabilities bitmap for the process:

    CapPrm:	00000000aa0435fb
    CapEff:	00000000aa0435fb


Compare the capabilities of the two Containers:

    00000000a80425fb
    00000000aa0435fb

In the first container, the capabilities for NET\_ADMIN and SYS\_TIME are not enabled. But in the second container, the capabilities for NET\_ADMIN and SYS\_TIME are enabled. The capability constants can be found in the capability.h file.

**Note:** When listing capabilities in the container manifest, use the name of the capability constant without the "CAP\_" prefix. For example, instead of listing "CAP\_SYS\_TIME", list "SYS\_TIME". This applies to all Linux capability constants, which are in the format of "CAP\_XXX".

## Set the Seccomp Profile for a Container
To configure the Seccomp profile for a Container, include the seccompProfile field in the securityContext section of the Pod or Container manifest. This field is a SeccompProfile object which includes type and localhostProfile options. The type option can be set to RuntimeDefault, Unconfined, or Localhost. 

If type is set to Localhost, the localhostProfile field must be set, which indicates the path of the pre-configured profile on the node, relative to the kubelet's Seccomp profile location. Here's an example that sets the Seccomp profile to the default profile of the node's container runtime.


    **securityContext**:
    ` `**seccompProfile**:
    `   `**type**: RuntimeDefault

Here is an example that sets the Seccomp profile to a pre-configured file at <kubelet-root-dir>/seccomp/my-profiles/profile-allow.json:


    **securityContext**:
    ` `**seccompProfile**:
    `   `**type**: Localhost
    `   `**localhostProfile**: my-profiles/profile-allow.json

## Assign SELinux labels to a Container
In order to assign SELinux labels to a Container, you need to include the seLinuxOptions field in the securityContext section of your Pod or Container manifest. The seLinuxOptions field is an SELinuxOptions object that enables you to set the SELinux labels for the container. An example of this is shown in the given configuration file, which sets the SELinux level for the container.


    **securityContext**:
    ` `**seLinuxOptions**:
    `   `**level**: "s0:c123,c456"

**Note:** To assign SELinux labels, the SELinux security module must be loaded on the host operating system.
### Efficient SELinux volume relabeling

By default, the container runtime assigns SELinux labels to all files on all Pod volumes recursively. To speed up this process, Kubernetes can use a mount option -o context=<label> to instantly change the SELinux label of a volume. In order for this to work, the following conditions must be met:

- Alpha feature gates ReadWriteOncePod and SELinuxMountReadWriteOncePod must be enabled.
- The Pod must use a PersistentVolumeClaim with accessModes ["ReadWriteOncePod"]
- The Pod or all its Containers that use the PersistentVolume claim must have seLinuxOptions set.
- The corresponding PersistentVolume must be either a volume that uses a CSI driver or a volume that uses the legacy iscsi volume type.
- If using a volume backed by a CSI driver, the CSI driver must announce support for mounting with -o context by setting spec.seLinuxMount: true in its CSIDriver instance.
- For other volume types, SELinux relabelling happens through recursively changing the SELinux label for all inodes in the volume, which can take longer for volumes with more files and directories.

**Note:** In version 1.25 of Kubernetes, if the kubelet is restarted, it can forget the labels assigned to volumes. This can result in errors when starting Pods that claim there are conflicting SELinux labels, even though there are none. To avoid this issue, it's important to fully drain nodes before restarting the kubelet.

## Discussion
The security context for a Pod applies to the Pod's Containers and also to the Pod's Volumes when applicable. Specifically, when fsGroup and seLinuxOptions are specified in the security context, the following actions will be taken on the Volumes:

- fsGroup: Volumes that support ownership management will have their ownership and write permissions set to the GID specified in fsGroup.
- seLinuxOptions: Volumes that support SELinux labeling will be relabeled to match the label specified under seLinuxOptions, usually only the level section needs to be set, this sets the Multi-Category Security (MCS) label for all Containers in the Pod as well as the Volumes.

**Warning:** When specifying the Multi-Category Security (MCS) label for a Pod, all Pods that have the same label will have access to the Volume. If you want to limit access and provide protection between Pods, it's necessary to assign a unique MCS label to each Pod.


