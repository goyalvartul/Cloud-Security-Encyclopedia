
# Elevate Your AKS Security: Boosting Authentication and Authorization - Part 3

## Authorization in Kubernetes

Similar to the process of Authentication, when setting up a Kubernetes cluster, the API server can be configured to use one or more Authorization modules. These include:

- **ABAC**: policies defined in a static file
- **RBAC**: policies that can be dynamically added or removed using Kubernetes objects. Kubernetes RBAC is a built-in role-based access control mechanism that allows you to create and assign roles (sets of permissions) for any object or type of object within the cluster.
- **Webhook**: policy decisions delegated to an external HTTP(S) service.
- **Node**: specific to authorize API requests made by kubelets.

You can also choose to allow or deny all requests. To learn more about configuring the Kube-API server for both authentication and authorization, check out the list of flags that can be passed when deploying the API server.

The webhook authorization module enables Kubernetes authorization to work with existing organization-wide or cloud-provider-wide access control systems. AKS relies on this mechanism to integrate Kubernetes authorization with Azure RBAC, ensuring that the request issuer has permission at either level to access Kubernetes cluster resources.

### Authorization in AKS

In AKS, there are two options for authorization mode: Kubernetes RBAC and Azure RBAC. When both are enabled, authorization can be granted at either level for user accounts, service principals, and groups created in Azure AD. However, for identities that exist only within the cluster, such as regular Kubernetes users and Kubernetes service accounts, access can only be granted using Kubernetes RBAC.

It is highly recommended to enable both authorization modes as this approach allows for unified management and access control across Azure resources, AKS, and Kubernetes resources. By doing so, it ensures that user accounts, service principals, and groups created in Azure AD can be granted access at either level, while identities that exist only within the cluster, such as regular Kubernetes users and Kubernetes service accounts, can only be granted access using Kubernetes RBAC.

In this blog, we have discussed the concept of identity and the different options available for authentication and authorization in both Kubernetes and AKS. To summarize, in the reference implementation for AKS, Azure recommends using Azure AD-integrated AKS clusters with local accounts disabled for authentication. Both Azure RBAC and Kubernetes RBAC should be enabled to operate the cluster. Azure RBAC should be used to authorize users and groups while Kubernetes RBAC should be used to authorize Kubernetes service accounts.

It is crucial to respect the principle of Least Privilege at each authorization level to make sure that every user or application can only access the information and resources that are necessary for its legitimate purpose. This will help to safeguard the security of the cluster and protect sensitive data.
