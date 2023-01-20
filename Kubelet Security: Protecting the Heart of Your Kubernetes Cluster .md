# Kubelet Security: Protecting the Heart of Your Kubernetes Cluster 


The kubelet is a crucial component in any Kubernetes deployment, often referred to as the "Kubernetes agent" software. It plays a key role in connecting the nodes to the cluster logic. Its main purpose is to manage the local container engine (such as Docker) and ensure that pods described in the API are created, run, maintained in a healthy state, and eventually destroyed when necessary.

Two communication interfaces are relevant to the kubelet:

- Access to the Kubelet REST API, which is used by users or software (usually just the Kubernetes API entity).
- Kubelet binary accessing the local Kubernetes node and the Docker engine.



![](images/image6.png)


The two interfaces of kubelet are secured by default through:

- Security-related configuration options passed to the kubelet binary
- NodeRestriction admission controller
- Role-Based Access Control (RBAC) to access the kubelet API resources.



### **Kubelet security – access to the kubelet API**

The kubelet security configuration options are typically passed as arguments to the kubelet binary. However, for newer versions of Kubernetes (1.10+), it's also possible to use a kubelet configuration file. Regardless of the method used, the syntax for the parameters remains the same. 

As an example, here is a configuration:

    /home/kubernetes/bin/kubelet –v=2 –kube-reserved=cpu=70m,memory=1736Mi –allow-privileged=true –cgroup-root=/ –pod-manifest-path=/etc/kubernetes/manifests –experimental-mounter-path=/home/kubernetes/containerized\_mounter/mounter –experimental-check-node-capabilities-before-mount=true –cert-dir=/var/lib/kubelet/pki/ –enable-debugging-handlers=true –bootstrap-kubeconfig=/var/lib/kubelet/bootstrap-kubeconfig –kubeconfig=/var/lib/kubelet/kubeconfig –anonymous-auth=false –authorization-mode=Webhook –client-ca-file=/etc/srv/kubernetes/pki/ca-certificates.crt –cni-bin-dir=/home/kubernetes/bin –network-plugin=cni –non-masquerade-cidr=0.0.0.0/0 –feature-gates=ExperimentalCriticalPodAnnotation=true

When configuring the kubelet parameters, it's important to verify the following Kubernetes security settings:

- anonymous-auth is set to false to disable anonymous access, this will send 401 Unauthorized responses to unauthenticated requests.
- The --client-ca-file flag is present, providing a CA bundle to verify client certificates.
- --authorization-mode is not set to AlwaysAllow, as the more secure Webhook mode delegates authorization decisions to the Kubernetes API server.
- --read-only-port is set to 0 to prevent unauthorized connections to the read-only endpoint (optional).



### **Kubelet security – kubelet access to Kubernetes API**

As previously discussed, the level of access granted to a kubelet is determined by the NodeRestriction Admission Controller on RBAC-enabled versions of Kubernetes (stable from version 1.8+). 

Kubelets are bound to the system:node Kubernetes clusterrole. 

If the NodeRestriction is enabled in your API, kubelets will only be able to modify their own Node API object and the Pod API objects that are bound to their node. This is a static restriction currently. 

To check if this admission controller is enabled, you can check the Kubernetes nodes by running the apiserver binary.

    $ **ps** aux | **grep** apiserver | **grep** admission-control

    --admission-control=Initializers,NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,ResourceQuota



### **RBAC example, accessing the kubelet API with curl**

Normally, only the Kubernetes API server needs to use the kubelet REST API. As previously stated, this interface needs to be protected as it allows the execution of arbitrary pods and commands on the hosting node.

You can test communication with the kubelet API directly from the node shell:

    curl  -k https://localhost:10250/pods
    Forbidden (user=system:anonymous, verb=get, resource=nodes, subresource=proxy)


The kubelet uses RBAC for authorization and the response indicates that the default anonymous system account is not permitted to connect.

To contact the secure port, you need to impersonate the API server kubelet client:


    curl --cacert /etc/kubernetes/pki/ca.crt --key /etc/kubernetes/pki/apiserver-kubelet-client.key --cert /etc/kubernetes/pki/apiserver-kubelet-client.crt -k https://localhost:10250/pods | jq .

    {
    `  `"kind": "PodList",
    `  `"apiVersion": "v1",
    `  `"metadata": {},
    `  `"items": [
    `    `{
    `      `"metadata": {
    `        `"name": "kube-controller-manager-kubenode",
    `        `"namespace": "kube-system",



The port numbers may differ depending on the method of deployment and initial configuration.

