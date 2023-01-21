
﻿# Kuber-Proofing: Secure Your Pods with These Proven Tips for Configuring a Security Context - Part 1

A security context is a set of settings that determine the level of privilege and access control for a Pod or Container. These settings include but are not limited to:

- Discretionary Access Control: Determining the permission to access an object, like a file, based on user ID (UID) and group ID (GID).
- Security Enhanced Linux (SELinux): Assigning security labels to objects.
- Specifying whether the container runs as privileged or unprivileged.
- Linux Capabilities: Granting specific privileges to a process, but not all privileges of the root user.
- AppArmor: Restricting the capabilities of individual programs using program profiles.
- Seccomp: Filtering system calls for a process.
- allowPrivilegeEscalation: Controlling whether a process can gain more privileges than its parent process. This setting directly controls whether the no\_new\_privs flag is set on the container process, and is always true when the container:
- is run as privileged, or
- has CAP\_SYS\_ADMIN
- readOnlyRootFilesystem: Mounting the container's root filesystem as read-only.



## Before you begin

In order to complete this tutorial, you must have a Kubernetes cluster set up and the kubectl command-line tool configured to communicate with it. It is recommended to have a cluster with at least two nodes that are not used as control plane hosts. If you do not have a cluster, you can create one using minikube or any other Kubernetes playground. 

To check the version of kubectl, enter "kubectl version" in the command line.



## Set the security context for a Pod

To configure security settings for a Pod, include the securityContext field in the Pod's configuration file. The securityContext field is a PodSecurityContext object that holds the security settings for the Pod. These settings apply to all the Containers within the Pod. In this example, a Pod's configuration file is provided that includes both a securityContext and an emptyDir volume.


pods/security/security-context.yaml 

    **apiVersion**: v1
    **kind**: Pod
    **metadata**:
    ` `**name**: security-context-demo
    **spec**:
    ` `**securityContext**:
    `   `**runAsUser**: 1000
    `   `**runAsGroup**: 3000
    `   `**fsGroup**: 2000
    ` `**volumes**:
    ` `- **name**: sec-ctx-vol
    `   `**emptyDir**: {}
    ` `**containers**:
    ` `- **name**: sec-ctx-demo
    `   `**image**: busybox:1.28
    `   `**command**: [ "sh", "-c", "sleep 1h" ]
    `   `**volumeMounts**:
    `   `- **name**: sec-ctx-vol
    `     `**mountPath**: /data/demo
    `   `**securityContext**:
    `     `**allowPrivilegeEscalation**: **false**


The configuration file outlines security settings for a Pod in Kubernetes. The runAsUser field sets the user ID for all processes within the Pod's containers to 1000, while the runAsGroup field sets the primary group ID to 3000. Omitting this field would default the primary group ID to root (0). The fsGroup field specifies that the container's processes should also be part of group ID 2000, and any files created in the /data/demo volume will be owned by this group.

Create the Pod:

    kubectl apply -f https://k8s.io/examples/pods/security/security-context.yaml

Verify that the Pod's Container is running:

    kubectl get pod security-context-demo

Get a shell to the running Container:

    kubectl exec -it security-context-demo -- sh

In your shell, list the running processes:

    ps

The output shows that the processes are running as user 1000, which is the value of runAsUser:

    PID   USER     TIME  COMMAND
    1       1000      0:00      sleep 1h
    6       1000      0:00      sh



In your shell, navigate to /data, and list the one directory:

    cd /data

    ls -l

The output shows that the /data/demo directory has group ID 2000, which is the value of fsGroup.

    drwxrwsrwx 2 root 2000 4096 Jun  6 20:08 demo

In your shell, navigate to /data/demo, and create a file:

    cd demo

    echo hello > testfile

List the file in the /data/demo directory:

    ls -l

The output shows that testfile has group ID 2000, which is the value of fsGroup.

    -rw-r--r-- 1 1000 2000 6 Jun  6 20:08 testfile

Run the following command:

    id

The output is similar to this:

    uid=1000 gid=3000 groups=2000

From the output, you can see that gid is 3000 which is same as the runAsGroup field. If the runAsGroup was omitted, the gid would remain as 0 (root) and the process will be able to interact with files that are owned by the root(0) group and groups that have the required group permissions for the root (0) group.

Exit your shell:

    exit

