# Elevate Your AKS Security: Boosting Authentication and Authorization - Part 1


Kubernetes does not have a built-in way to handle user management, but it allows administrators to use their preferred identity service provider. This prevents the need to duplicate user information and management. Azure Kubernetes Service offers integration with Azure AD, a robust identity management solution, for managing accounts and security.

This article will explore the concept of identity in depth, including how it is established and the various authentication and authorization options available in both Kubernetes and Azure Kubernetes Service.

Currently, there are four methods available to set up Authentication and Authorization in an AKS cluster.

## **1- Authentication using local accounts for both user and admin access / Authorization using Kubernetes RBAC (default)**



![](images/image39.png)


Users and administrators obtain a X509 client certificate that is specific to the cluster, known as local accounts. The certificate is created with the common name "master client" and belongs to the "system:masters" group. It is also bound to the "cluster-admin" role, which grants the user full access to the AKS cluster.

Since there is no identity management system integrated with the cluster, to create additional users, administrators must manually create a certificate for each user, signed by the cluster's certificate authority. Then, using Kubernetes RBAC, users can be authorized to perform operations on cluster resources. For instructions on how to create a user, refer to the Kubernetes documentation.

An example command to create a cluster with this setup:

    az aks create -g MyResourceGroup -n MyManagedCluster


## **2- Authentication using local accounts for both user and admin access/ no authorization mechanism is enabled**


![](images/image40.png)

Similar to the previous method for authentication, this method also uses client certificates specific to the cluster as local accounts. However, no authorization mechanism is enabled, so any authenticated user will have full administrator access to the cluster.

An example command to create a cluster with this setup:

    az aks create -g MyResourceGroup -n MyManagedCluster \

`   `**--disable-rbac**`                      



## **3- Authentication using Azure Active directory / Authorization using Kubernetes RBAC only**



![](images/image41.png)

For this type of cluster, users are directed to Azure Active Directory for authentication. Then, Kubernetes role-based access control is used as the sole authorization method within the cluster.

To set up this type of cluster, an Active Directory group must be configured to act as the administrator for the cluster. Users who belong to this group can then configure access control for other individual AD users or groups by assigning them Kubernetes roles and cluster roles.

An example command to create a cluster with this setup is:

Creating an Azure AD group that will contain admin users for the cluster:

    \# create admin group
    az ad group create --display-name myAKSAdminGroup --mail-nickname myAKSAdminGroup
    \# get the group object ID
    admin\_group\_object\_id=$(az ad group show --group myAKSAdminGroup --query objectId -o tsv)

You can use the following command to add admin users to the group:


    az ad group member add \
    `    `--group myAKSAdminGroup \
    `    `--member-id [[put your account object id here]]

Create the cluster while specifying the admin group for the cluster


    az aks create -g MyResourceGroup -n MyManagedCluster \
    `   `**--enable-aad \**
    `   `**--aad-admin-group-object-ids $admin\_group\_object\_id \**
    `   `**--disable-local-accounts # optional (disable local accounts)**


Note: Local accounts are enabled by default. To enable centralized user access control and governance through Azure AD, local accounts must be disabled. This feature is currently in preview, refer to the official documentation for instructions on how to enable it.

To make the group an administrator for the cluster, AKS will create a cluster role binding that assigns it a cluster role that grants full administrator access to the cluster. To confirm this, follow these steps:

First, retrieve the cluster credentials:

**az aks get-credentials -g MyResourceGroup -n MyManagedCluster**

Check the cluster role binding created by AKS and observe how it references the admin group as the subject (you may be prompted to authenticate first).

kubectl get clusterrolebinding aks-cluster-admin-binding-aad -o yaml

    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    Metadata:
    `     `name: aks-cluster-admin-binding-aad
    `     `resourceVersion: "362"
    `     `uid: f1cb3b3e-3960-4946-a168-1f2cb7ebb4b0
    roleRef:
    ` `apiGroup: rbac.authorization.k8s.io
    ` `kind: **ClusterRole**
    ` `name: **cluster-admin**
    Subjects:
    \- apiGroup: rbac.authorization.k8s.io
    ` `kind: **Group**
    ` `name: **c6f05369-71f1-4cb1-b72f-bf06071ba539 # admin group object ID**


## **4- Authentication using Azure Active directory / Authorization using both kubernetes RBAC and Azure RBAC (recommended):**

![](images/image42.png)


Users authenticate to the cluster using Azure Active Directory. In this configuration, the Kubernetes API will delegate access control (authorization) to a Kubernetes Authorization webhook server that is deployed and managed as part of the AKS cluster. This webhook will authorize requests based on Azure RBAC and Kubernetes RBAC if the identity exists within Azure AD, and Kubernetes RBAC if the identity is local to the cluster.

Since Azure RBAC can be used to authorize Azure AD accounts to perform actions on the cluster, there is no need to configure an admin group for the cluster in this case.

Example command to create a cluster with this setup:

    az aks create -g MyResourceGroup -n MyManagedCluster \
    `   `**--enable-aad \
    `   `--enable-azure-rbac \
    `   `--disable-local-accounts # optional (disable local accounts)**



Note: Local accounts are enabled by default. To enable centralized user access control and governance through Azure AD, local accounts must be disabled. This feature is currently in preview, refer to the official documentation for instructions on how to enable it.

After creating the cluster, you can assign Azure AD principals and Azure RBAC roles that can be scoped to specific namespaces or across the entire AKS cluster as needed.

For instance, to make yourself an administrator of the entire cluster, you can assign yourself the "Azure Kubernetes Service RBAC Cluster Admin" role using the following commands:

    \# Get your AKS Resource ID
    AKS\_ID=$(az aks show -g MyResourceGroup -n MyManagedCluster --query id -o tsv)
    \# replace AAD-ENTITY-ID with your account email
    az role assignment create --role "Azure Kubernetes Service RBAC Cluster Admin" --assignee <YOUR-AAD-ENTITY-ID> --scope $AKS\_ID



