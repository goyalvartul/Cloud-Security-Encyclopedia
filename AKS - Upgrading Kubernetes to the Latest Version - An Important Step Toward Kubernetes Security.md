# AKS - Upgrading Kubernetes to the Latest Version - An Important Step Toward Kubernetes Security


Throughout the process of managing your application and cluster, it may be necessary to upgrade to the most recent version of Kubernetes. Upgrading your Azure Kubernetes Service (AKS) cluster can be accomplished through utilizing the Azure CLI, Azure PowerShell, or the Azure portal.

In this seventh installment of a tutorial series, you will learn how to:

- Determine the current and available versions of Kubernetes.
- Upgrade the Kubernetes nodes.
- Verify that the upgrade was completed successfully.

## **Get available cluster versions**


Prior to upgrading a cluster, use the Get-AzAksCluster command to verify the version of Kubernetes currently in use, as well as the region it is located in.

Run the following command on Azure Powershell

    Get-AzAksCluster -ResourceGroupName myResourceGroup -Name myAKSCluster |
    `  `Select-Object -Property Name, KubernetesVersion, Location

In the following example output, the current version is *1.19.9*

Output



    Name            KubernetesVersion       Location
    \----            -----------------       --------
    myAKSCluster    1.19.9                  eastus

To check which upgrade releases of Kubernetes are available in the region where your AKS cluster is located, use the Get-AzAksVersion command.

Run the following command on Azure Powershell

    Get-AzAksVersion -Location eastus | Where-Object OrchestratorVersion -gt 1.19.9

The available versions are shown under *OrchestratorVersion*.

Output


    Default     IsPreview     OrchestratorType     OrchestratorVersion
    \-------     ---------     ----------------     -------------------
    `                          `Kubernetes           1.20.2
    `                          `Kubernetes           1.20.5


**Upgrading a Cluster**

During the upgrade process, AKS nodes are handled with care to minimize disruptions to running applications. The following steps take place:

- Kubernetes scheduler prevents further pods from being scheduled on the node that is being upgraded.
- Pods running on the node are scheduled on other nodes in the cluster.
- A new node is created that runs the latest Kubernetes components.
- Once the new node is ready and joined to the cluster, the Kubernetes scheduler starts scheduling pods on the new node.
- The old node is deleted, and the cordon and drain process begins on the next node in the cluster.

It is important to note that if no patch is specified, the cluster will automatically upgrade to the latest GA patch of the specified minor version. For example, setting --kubernetes-version to 1.21 will result in the cluster upgrading to 1.21.9.

When upgrading using the alias minor version, only a higher minor version is supported, for example, updating from 1.20.x to 1.20 will not trigger an upgrade to the latest GA 1.20 patch, however, upgrading to 1.21 will trigger an upgrade to the latest GA 1.21 patch.

To upgrade your AKS cluster, use the Set-AzAksCluster command.

Run the following command on Azure Powershell

    Set-AzAksCluster -ResourceGroupName myResourceGroup -Name myAKSCluster -KubernetesVersion <KUBERNETES\_VERSION>

Note: You can only upgrade one minor version at a time. For example, you can upgrade from *1.14.x* to *1.15.x*, but you cannot upgrade from *1.14.x* to *1.16.x* directly. To upgrade from *1.14.x* to *1.16.x*, first upgrade from *1.14.x* to *1.15.x*, then perform another upgrade from *1.15.x* to *1.16.x*.

The following example output shows the result of upgrading to *1.19.9*. Notice the *kubernetesVersion* now reports *1.20.2*.

Output

    ProvisioningState       : Succeeded
    MaxAgentPools           : 100
    KubernetesVersion       : 1.20.2
    PrivateFQDN             :
    AgentPoolProfiles       : {default}
    Name                    : myAKSCluster
    Type                    : Microsoft.ContainerService/ManagedClusters
    Location                : eastus
    Tags                    : {}


## **View the upgrade events**

During the process of upgrading your cluster, the following Kubernetes events may take place on the nodes:

- Surge: A surge node is created.
- Drain: Pods are evicted from the node. Each pod has a 5-minute time frame to complete the eviction.
- Update: The update of a node has been successful or unsuccessful.
- Delete: A surge node is deleted.

To view events in the default namespaces while running an upgrade, use the command kubectl get events.

Run the following on on Azure CLI

    kubectl get events 

The example output below displays some of the aforementioned events that may occur during an upgrade.

**Output**

    default 2m1s Normal Drain node/aks-nodepool1-96663640-vmss000001 Draining node: [aks-nodepool1-96663640-vmss000001]
    
    default 9m22s Normal Surge node/aks-nodepool1-96663640-vmss000002 Created a surge node [aks-nodepool1-96663640-vmss000002 nodepool1] for agentpool %!s(MISSING


## **Validate an upgrade**

To ensure that the upgrade has been completed successfully, use the Get-AzAksCluster command.

Run the following on Azure PowerShell

    Get-AzAksCluster -ResourceGroupName myResourceGroup -Name myAKSCluster |
    `  `Select-Object -Property Name, Location, KubernetesVersion, ProvisioningState

The example output below demonstrates that the AKS cluster is running the KubernetesVersion 1.20.2.

Output

    Name         Location   KubernetesVersion   ProvisioningState
    \----         --------   -----------------   -----------------
    myAKSCluster eastus     1.20.2              Succeeded
