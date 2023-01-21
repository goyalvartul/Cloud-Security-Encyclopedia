# Elevate Your AKS Security: Boosting Authentication and Authorization - Part 2

A user's every request to Kubernetes is sent as an API call to the RESTful interface offered by the Kube-API server. This process is also used internally, when different components of Kubernetes such as the scheduler, kubelets, and kube-proxy, for instance, want to access or modify cluster state from Etcd for communication with one another, their requests must pass through the API server, which will handle the request on their behalf.

The Kube-API server carries out three primary consecutive tasks for each incoming request before altering the cluster state. These are: Authentication, Authorization, and Admission Control as illustrated in the accompanying graph. In the next section, we will elaborate on the concept of identity in k8s and how the API server handles Authentication and Authorization, both from a general standpoint and from the perspective of Azure Kubernetes service.


![](images/image43.png)



# **Identity in Kubernetes**

There are two types of identity in Kubernetes: User accounts and Service accounts. User accounts represent individual users and are global and unique across all namespaces. These accounts are not managed by Kubernetes and can only be referred to within the cluster, meaning they cannot be created, stored, or deleted through an API call. Access to cluster resources can only be granted or revoked for these accounts. 

User accounts are typically created by an organization's identity management solution and can also belong to groups that can be referred to within the cluster when granting access. Service accounts, on the other hand, are special types of accounts meant to represent the applications running inside pods. Service accounts are namespaced, meaning each account must belong to a specific namespace and can have the same name as long as they are created in different namespaces. 

Service accounts are actual Kubernetes objects that are managed by the cluster and can be created and used as an identity for pods running an application if they need to interact with the API server. Service account credentials are stored as Kubernetes secrets in the same namespace. Every namespace comes with a "default" service account created automatically when the namespace is created. When deploying a pod, the credentials of the default service account of the namespace where the pod is run will be mounted as a volume in the pod's filesystem, unless a new service account is created and explicitly indicated as the identity for the pod.

To create a service account:

    $ kubectl create serviceaccount mysa

    serviceaccount/mysa created

Notice how a secret was created for the new service account:

    $ kubectl get secrets

    NAME                                                TYPE                                 DATA      AGE
    default-token-lqdkc  kubernetes.io/service-account-token      3            16h

**mysa-token-r78xj**     **kubernetes.io/service-account-token  3      9m$**

    $ kubectl get secrets mysa-token-r78xj   -o yaml

    apiVersion: v1
    data:
    ` `ca.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURE.....
    ` `namespace: ZGVmYXVsdA==
    ` `**token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNklpSjkuD.....**
    kind: Secret

In Kubernetes, secrets are encoded in base64 by default, therefore, by decoding the secret token field, one can use that token to take on the identity of the service account and authenticate to the cluster.

$ TOKEN=$(kubectl get secret mysa-token-r78xj -o jsonpath='{.data.token}'| base64 --decode)

    $ kubectl get pods --token $TOKEN

Error from server (Forbidden): pods is forbidden: User **"system:serviceaccount:default:mysa"** cannot list resource "pods" in API group "" in the namespace "default"

An error occurs because we did not grant any permissions to the Service account yet, However, as you can see, the request is authenticated as the service account "system:serviceaccount:default:mysa".

# **Identity in AKS**
AKS clusters, which are managed by Azure, have the same identities as Kubernetes clusters: user accounts and service accounts. 

These user accounts are typically created and managed outside of Kubernetes, but can be authenticated with a valid certificate from the cluster's certificate authority. 

However, this approach poses a security risk as the kubeconfig file contains private keys and certificates that need to be distributed to users. 

To address this risk, AKS supports integration with Azure AD, allowing users to access OnPrem, Azure, and AKS resources without the need for certificate-based kubeconfig files. When using Azure AD integration, it's recommended to disable local accounts that use x509 certificates for authentication and use Azure AD as the central source for creating and managing users. 

Within Azure AD, there are two types of identities that can be used: Azure user accounts and Azure service principals, which can both authenticate to AKS clusters integrated with Azure AD. Creating a user in this context means creating a user or service principal in Azure AD.

    az ad user create --display-name mydisplay --password {password} --user-principal-name myuserprincipal

To create a service principal in Azure AD

    az ad sp create-for-rbac --name ServicePrincipalName

# **Authentication (authn)**

Authentication is the process of verifying the identity of the entity making a request. Given our discussion of the two types of identities in k8s/AKS and the methods for creating users, let's explore the options for authenticating in a basic Kubernetes cluster versus an AKS cluster.

# **Kubernetes Authentication**

When creating a Kubernetes cluster, you have the option to enable one or more Authenticator Modules from the available methods, such as:

- static passwords or tokens file
- Client Certificates
- Bootstrap Tokens
- OpenID Connect Tokens (OIDC)
- authentication webhook, etc.

Enabling multiple modules when deploying the kube-api server allows the API server to verify the identity of the request against each enabled authenticator until one succeeds.

When a user runs a kubectl command, the Kubernetes CLI will convert it into an HTTP request and include client configuration for the API server to use for authentication. Some examples of client config include:

- a username and password if the cluster is configured to use HTTP basic authentication.
- a user x509 certificate if the cluster is configured to trust certificates issued from a certificate authority (CA).
- a bearer token that belongs to a user from a preconfigured file holding a set of records “tokens,users,uid,groups” each representing a user, its token, user ID and the groups they belong to if any.
- a bearer token dynamically generated when you create a new service account.
- a JWT token generated by an OAuth2 provider. The provider can be configured as a trusted issuer for the API server level or for an external Auth webhook server used to delegate authentication to it. Azure Active Directory is an example of an OAuth2 provider.

**AKS authentication**

======================
In AKS, both x509 client certificates and JWT tokens generated using Azure AD are supported methods for user authentication.

By default, AKS uses local accounts based on x509 certificates for authentication. The kube-api server verifies that the certificate presented by the user is valid and signed by the cluster's trusted certificate authority, and the user ID is taken from the certificate's common name (e.g., "CN=bob"). However, disabling client certificates is not possible without rotating the cluster's certificate authority.

On the other hand, using Azure AD integrated clusters eliminates the need to separately create user identities for the cluster and manage private keys or certificates. AKS achieves this by delegating authentication to an auth webhook server deployed and configured as part of the AKS cluster. The auth webhook server validates JWT tokens retrieved by users after authenticating to Azure AD and queries the MS Graph API for any group memberships the user has. The authentication flow is well-documented in Azure documentation.



![](images/image44.png)


To better understand how authentication works in AKS, let's examine an example of a kubeconfig file generated when creating an AKS cluster with Azure AD integration.

` `To follow along, you can create an Azure AD-enabled AKS cluster using option 3 or 4 from the instructions provided. The command "az aks get-credentials -g MyResourceGroup -n MyManagedCluster" generates a kubeconfig containing the configuration to access your newly created cluster, which we can then examine.

    $ kubectl config view
    apiVersion: v1
    Clusters:
    `  `- cluster:
    `      `certificate-authority-data: DATA+OMITTED
    `   `**server: <https://mymanagedc-myresourcegroup-2d2990-38887617.hcp.eastus.azmk8s.io:443>**
    ` `name: MyManagedCluster
    Contexts:
    `   `- context:
    `   `cluster: MyManagedCluster
    `   `user: clusterUser\_MyResourceGroup\_MyManagedCluster
    ` `name: MyManagedCluster
    **current-context: MyManagedCluster**
    kind: Config
    preferences: {}
    Users:
    \- name: clusterUser\_MyResourceGroup\_MyManagedCluster
    ` `**User:
    `   `Auth-provider:**
    `     `Config:
    `       `apiserver-id: 6dae42f8-4368-4678-94ff-3960e28e3630
    `       `client-id: 80faf920-1908-4b52-b5ef-a8e7bedfc67a
    `       `config-mode: "1"
    `       `environment: AzurePublicCloud
    `       `tenant-id: 72f988bf-86f1-41af-91ab-2d7cd011db47
    `     `name: azure

The kubeconfig file includes information such as the endpoint of the AKS API server, contexts for accessing different Kubernetes clusters added to the config, the current active context by default, and a section for users with the config needed to authenticate to the AKS application created in Azure AD.

When you first execute a command against the cluster, kubectl will use the Azure AD client application to sign you in using the oauth2 device authorization grant flow. To see the values added to the kubeconfig after authentication, you can execute a command against the cluster and follow the steps to authenticate to your account.

$ kubectl get nodes

To sign in, open the website[ ](https://microsoft.com/devicelogin)<https://microsoft.com/devicelogin> in a web browser and enter the code AVE3KTRBC to complete the authentication process. After successful authentication, examine the user section of the config file again.

    $ kubectl config view
    apiVersion: v1
    Clusters:
    \- cluster:
    `   `certificate-authority-data: DATA+OMITTED
    `   `server: https://mymanagedc-myresourcegroup-2d2990-38887617.hcp.eastus.azmk8s.io:443
    ` `name: MyManagedCluster
    Contexts:
    \- context:
    `   `cluster: MyManagedCluster
    `   `user: clusterUser\_MyResourceGroup\_MyManagedCluster
    ` `name: MyManagedCluster
    current-context: MyManagedCluster
    kind: Config
    preferences: {}
    Users:
    \- name: clusterUser\_MyResourceGroup\_MyManagedCluster
    ` `User:
    `   `Auth-provider:
    `     `Config:
    `       `**access-token: eyJ0eXAiOiJKV1QiLCJhbGciOiJ..........**
    `       `apiserver-id: 6dae42f8-4368-4678-94ff-3960e28e3630
    `       `client-id: 80faf920-1908-4b52-b5ef-a8e7bedfc67a
    `       `config-mode: "1"
    `       `environment: AzurePublicCloud
    `       `**expires-in: "3599"
    `       `expires-on: "1632903590"
    `       `refresh-token: 0.ARoAv4j5cvGGr0GRqy180BHbR.........**
    `       `tenant-id: 72f988bf-86f1-41af-91ab-2d7cd011db47
    `     `name: azure

Observe the four new fields that have been included following the authentication process: access-token, expires-in, expires-on, and refresh-token. The access token, which is utilized by the auth webhook to identify the user, can be decoded to view the information it contains. The access token is in the form of a JWT (JSON Web Token) and can be decoded using a tool such as JWT.io. The decoded output can be seen in the accompanying graph.



![](images/image45.png)

The JWT token contains information about the issuer, the User Principal name, and other details, as well as a signature at the end. This information is used by the webhook auth module to verify that the token was issued by Azure AD and that the message has not been tampered with.

# **Authorization (authz)**
After confirming the identity of the request, the kube-API proceeds to determine if the requester has the required permission to execute the requested action. This process is known as Authorization.
