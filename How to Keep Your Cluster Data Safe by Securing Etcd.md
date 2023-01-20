# How to Keep Your Cluster Data Safe by Securing Etcd


Deploying containers may seem simple, but managing them at a large scale, particularly when working with microservices and multiple cloud providers, requires careful planning. 

This is where an orchestration tool such as Kubernetes comes in. Kubernetes, which means "helmsman" in Greek, serves as a guide to navigate the complexities of containerized workloads and services by providing both configuration and automation. 

To function properly, Kubernetes relies on a helper called etcd.


## What is etcd?
Etcd is a distributed key-value store that is widely used for managing configuration data, state data, and metadata for Kubernetes. It is well-regarded for its many benefits, including:

- Every node in an etcd cluster has access to the full data store, ensuring all nodes are **fully replicated**.
- It is **highly available**, meaning it can handle hardware failures and network partitions without experiencing a single point of failure.
- It is also **reliably consistent**, ensuring that every data read is up to date and returns the latest data write across all clusters.
- It is **fast**, able to support up to 10,000 writes per second.
- It is **simple** to use, allowing any application, such as simple web apps or complex container orchestration engines, to read or write data to etcd using standard HTTP/JSON tools.
- It is **secure**, supporting automatic transport layer security (TLS) and optional secure socket layer (SSL) client certificate authentication.

In this post, we will focus on the security part.

## Why we need to secure etcd

Etcd plays a key role in hosting Kubernetes-related data and may contain sensitive information such as access credentials and private keys associated with digital certificates. These credentials are lucrative targets for malicious actors and should always be protected. 

Compromised or stolen credentials are the key source of successful data breaches. Additionally, they can be used to impersonate legitimate entities or create rogue certificates to access application data and spread malware.

Traditionally, this sensitive and crucial data was not stored in a centralized manner and was owned by established teams responsible for data protection and safeguarding. However, in Kubernetes platforms, these credentials are now stored in etcd. Because of the sensitivity and how critical the data is, it is important to harden the etcd store.

The Center for Internet Security (CIS) has developed benchmarks for hardening and securing Kubernetes. These guidelines suggest using TLS-based authentication to allow (or not) access to etcd. 

In the words of CIS:

“Its access should be restricted to specifically designated clients and peers only. Authentication to etcd is based on whether the certificate presented was issued by a trusted certificate authority. There is no checking of certificate attributes such as common name or subject alternative name. As such, if any attackers were able to gain access to any certificate issued by the trusted certificate authority, they would be able to gain full access to the etcd database.”

However, using TLS authentication is not sufficient. Private keys can be compromised, and attackers can impersonate authorized users to steal data. Hence, there is a need for a defense-in-depth approach. Except for access control, we must encrypt the data at rest in etcd.

## Compliance requirements and encryption
Security and privacy regulations such as PCI DSS, HIPAA, and GDPR require that certain data be protected through encryption.

The HIPAA Security Rule states that electronic protected health information (ePHI) must be made unreadable to unauthorized individuals through encryption using a confidential process or key that is not easily breached. This can be achieved through the use of an algorithmic process that transforms the data into a form that is difficult to interpret without the use of a confidential process or key. To protect against a potential breach, the decryption tools should be stored separately from the encrypted data.

StackRox points out that the data stored in etcd in the OpenShift Container Platform is not encrypted by default. However, they recommend enabling etcd encryption to enhance data security within the cluster.

According to the RedHat OpenShift documentation page, encryption of data at rest in etcd will secure the following server resources:

- Secrets
- ConfigMaps
- Routes
- OAuth access tokens
- OAuth authorized tokens

