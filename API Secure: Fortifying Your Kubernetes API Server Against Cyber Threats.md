# API Secure: Fortifying Your Kubernetes API Server Against Cyber Threats


The control structure in a Kubernetes cluster involves the Control Plane managing Nodes, which in turn manage Pods, which control containers, and these containers manage applications. However, who manages the Control Plane? Securing access to the Kubernetes API, which enables management of the entire cluster, is a crucial aspect of Kubernetes security. 

The Kubernetes hardening guide by the NSA advises using strong authentication and authorization to limit access and reduce the potential attack surface. This guide focuses on providing best practices and instructions for securing API access control in a Kubernetes cluster, including implementing strong authentication and authorization. 

Even if using managed Kubernetes services like AWS EKS or GCP Kubernetes engine, this guide can aid in understanding the internal workings of access control and aid in overall Kubernetes security planning.

## **Why is Kubernetes API access control important for Kubernetes security?**

It is important to have an understanding of how Kubernetes functions internally before considering its API access control from a security perspective. Kubernetes is not a standalone product, rather it is a combination of multiple programs that work together. This means that when considering potential attack surfaces, it is necessary to take into account the entire collection of tools and components in Kubernetes, rather than just a single unit as in other software or products.


![](images/iamge13.png)


Additionally, the 4C's of Cloud Native security model, which advocates for a four-layer approach to secure cloud computing resources, highlights that Kubernetes is one of the layers (cluster). Weaknesses and errors in configuration in any of the layers can create a security risk for the other layers, which means that the security of Kubernetes is interconnected with the security of the overall Cloud Native components.

## **5 Kubernetes API network access best practices**

API access control in Kubernetes follows a three-step process that involves authenticating the request, checking for proper authorization, performing request admission control, and finally granting access. However, prior to beginning the authentication process, it is important to ensure that network access control and TLS connections are set up properly. 

Listed below are the four recommended best practices for securing direct network access to the Kubernetes API (control plane).

### **1. Configure and use TLS everywhere**|

TLS connections should be used for all connections to the API server, communication within the Control Plane, and communication between the Control Plane and Kubelet. Though the API server can easily be configured to use TLS by supplying --tls-cert-file=[file] and --tls-private-key-file=[file] flag to the kube-apiserver,, managing certificates within a Kubernetes cluster, which can quickly scale up or down, can be difficult. To address this, Kubernetes has implemented a feature called TLS bootstrapping, which allows for automatic signing and configuration of TLS certificates within the cluster.

### **2. Harden global TLS connection settings**

To configure the kube-apiserver with flags related to Transport Layer Security (TLS), you can use the following options:

- --secure-port: This flag specifies the network port that should be used for HTTPS connections to the kube-apiserver. The default is 6443, but you can change it to a different port number by providing the desired port number with this flag.
- --tls-cert-file, --tls-private-key-file: These flags configure x509 certificates and a private key for HTTPS connections.
- --cert-dir: This flag configures the directory where the TLS cert and key files are stored. The --tls-cert-file and --tls-private-key-file flags will be given priority over the certificates in this directory. The default location for the --cert-dir is /var/run/kubernetes.
- --tls-cipher-suites: This flag configures the preferred cipher suites for TLS. If not supplied, the kube-apiserver runs with the default cipher suites offered by Golang.
- --tls-min-version: This flag configures the minimum supported version of TLS. The possible values are: VersionTLS10, VersionTLS11, VersionTLS12, VersionTLS13.
- --tls-sni-cert-key: This flag configures Server Name Indication (SNI) as a pair value, --tls-sni-cert-key=testdomain.crt,testdomain.key.
- --strict-transport-security-directives: This flag configures HTTP Strict Transport Security (HSTS).
- --requestheader-client-ca-file, --proxy-client-cert-file, --proxy-client-key-file: Kubernetes allows for extending the kube-apiserver with custom APIs using an aggregation layer. To secure communication between the kube-apiserver and the custom API server, these flags let you configure x509 certificates for secure and trusted communication.

### **3. Harden TLS connection between API server and Kubelet**

The flags --kubelet-certificate-authority, --kubelet-client-certificate, and --kubelet-client-key allow you to set up the CA, client certificate, and client private key for secure communication between the Kubernetes API server and Kubelet using TLS.
### **4. TLS connection between API server and etcd:**

The flags --etcd-cafile, --etcd-certfile, and --etcd-keyfile enable you to establish secure connections using TLS between the API and etcd by configuring the CA file, certificate file, and key file respectively.
### **5. Secure direct network access to the API server**

- **Do not enable localhost port in production.**
  The Kubernetes API server uses HTTP protocol to serve on two ports by default: a localhost port and a secure port. The localhost port does not require encryption and requests made to it are not subject to authentication and authorization checks. It is important to note that the localhost port should not be accessible outside the testing environment of a Kubernetes cluster.**

- **Use Kubectl Proxy to manage secure client access.**
  To ensure secure communication through authentication and transport, proper management of secrets is crucial on both the client and server side. In a distributed team, multiple users may be using kubectl from different locations, increasing the risk of compromise of credentials such as cert files and tokens. To mitigate this risk, you can set up a Bastion server where kubectl proxy is configured, allowing users to send HTTPS requests to this Bastion server, which then forwards the requests to the API server with the required credentials. This way, the secure client credentials never leave the secure Bastion server. This method also prevents users from accessing the API server directly through network access.**

- **Know and secure API server proxy feature.
  Kubernetes has an in-built Bastion server within the API server that acts as a proxy to access services running on the cluster. The URL scheme for accessing these services is in the format of http://kubernetes\_master\_address/api/v1/namespaces/namespace-name/services/service-name[:port\_name]/proxy. For example, if the elasticsearch-logging service is running on the cluster, it can be accessed via the URL[ ](https://clusteripordomain/api/v1/namespaces/kube-system/services/elasticsearch-logging/proxy)[https://ClusterIPOrDomain/api/v1/namespaces/kube-system/services/elasticsearch-logging/proxy](https://clusteripordomain/api/v1/namespaces/kube-system/services/elasticsearch-logging/proxy).**

This feature allows access to internal services that may not be reachable from external networks. However, it can also be used by malicious users to access internal services they are not authorized to access. To allow administrative access to the API server but block access to internal services, you can use an HTTP proxy or a WAF to block requests to these endpoints.


**Use HTTP proxies, load balancers, and network firewalls.**

One way to enhance the security of the Kubernetes API is to use a simple HTTP proxy server, such as Nginx proxy, to quickly implement URL and security rules in front of the API. For maximum security, consider using a load balancer like AWS ELB or Google Cloud Load Balancer and network firewalls to control direct access to the Kubernetes API servers.
