# Kuber-Proofing: Secure Your Pods with These Proven Tips for Configuring a Security Context - Part 2

## Configure volume permission and ownership change policy for Pods

The fsGroupChangePolicy field within a securityContext can be used to control how Kubernetes manages ownership and permissions for a volume. This field only applies to volume types that support fsGroup controlled ownership and permissions and has two options:

- OnRootMismatch: Only changes ownership and permissions if the root directory's permissions do not match the expected permissions of the volume, potentially reducing the time needed to change ownership and permissions.
- Always: Changes ownership and permissions of the volume every time it is mounted.

For example:

    **securityContext**:
    ` `**runAsUser**: 1000
    ` `**runAsGroup**: 3000
    ` `**fsGroup**: 2000
    ` `**fsGroupChangePolicy**: "OnRootMismatch"

It's worth noting that the fsGroupChangePolicy field only applies to certain types of volumes and has no impact on temporary volumes like secret, configMap and emptydir.
## Delegating volume permission and ownership change to CSI driver

When you use a Container Storage Interface (CSI) driver that supports the VOLUME\_MOUNT\_GROUP NodeServiceCapability, the CSI driver, instead of Kubernetes, is responsible for setting file ownership and permissions based on the fsGroup defined in the securityContext. As a result, the fsGroupChangePolicy setting will not have an impact and the driver is expected to mount the volume with the fsGroup specified, making it readable and writable by that group.
## Set the security context for a Container
To set security settings for a Container, include the securityContext field in the Container's configuration file. The securityContext field is an object that controls the security settings for the Container. The settings specified for a Container only apply to that specific Container and will override any settings made at the Pod level if there are conflicts. However, the security settings for a Container will not affect the Pod's Volumes. Below is an example of a configuration file for a Pod with one Container, both of which have securityContext fields.

pods/security/security-context-2.yaml 

    **apiVersion**: v1
    **kind**: Pod
    **metadata**:
    ` `**name**: security-context-demo-2
    **spec**:
    ` `**securityContext**:
    `   `**runAsUser**: 1000
    ` `**containers**:
    ` `- **name**: sec-ctx-demo-2
    `   `**image**: gcr.io/google-samples/node-hello:1.0
    `   `**securityContext**:
    `     `**runAsUser**: 2000
    `     `**allowPrivilegeEscalation**: **false**


Create the Pod:

    kubectl apply -f https://k8s.io/examples/pods/security/security-context-2.yaml

Verify that the Pod's Container is running:

    kubectl get pod security-context-demo-2

Get a shell into the running Container:

    kubectl exec -it security-context-demo-2 -- sh

In your shell, list the running processes:

    ps aux

The output shows that the processes are running as user 2000. This is the value of runAsUser specified for the Container. It overrides the value 1000 that is specified for the Pod.

    USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    
    2000         1  0.0  0.0   4336   764 ?        Ss   20:36   0:00 /bin/sh -c node server.js
    
    2000         8  0.1  0.5 772124 22604 ?        Sl   20:36   0:00 node server.js

...

Exit your shell:

    exit

