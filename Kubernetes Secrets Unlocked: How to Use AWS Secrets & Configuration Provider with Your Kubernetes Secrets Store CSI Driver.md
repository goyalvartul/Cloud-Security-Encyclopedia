# Kubernetes Secrets Unlocked: How to Use AWS Secrets & Configuration Provider with Your Kubernetes Secrets Store CSI Driver


AWS Secrets Manager now enables you to securely retrieve secrets from AWS Secrets Manager for use in your Amazon Elastic Kubernetes Service (Amazon EKS) Kubernetes pods. With the launch of AWS Secrets and Config Provider (ASCP), you now have an easy-to-use plugin for the industry-standard Kubernetes Secrets Store and Container Storage Interface (CSI) driver, used for providing secrets to applications that operate on Amazon EKS. You can now use ASCP to provide compatibility for legacy Kubernetes workloads that fetched secrets through the file system or etcd. Previously, you had to store your secrets as plaintext in configuration files, or use encryption with Kubernetes etcd to securely see and access secrets through the filesystem. You also had to write custom code to rotate secrets, resulting in a maintenance challenge. To create secure access to your pods, you had to split up clusters to control access, increasing the operational load.

Now, with ASCP, you can securely store and manage your secrets in Secrets Manager, and retrieve them through your applications that are running on Kubernetes, without the need to write custom code. You also have the added benefit of using AWS Identity and Access Management (IAM) and resource policies on your secret to limit and restrict access to specific Kubernetes pods inside a cluster. This tightly controls which secrets are accessible by which pods. If you have enabled the rotation reconciler feature of the Secret Store CSI driver, ASCP will work with it to retrieve the latest secret from the secret provider. After ASCP is installed and enabled, it helps ensure that your applications always receive the most current version of the secret when the pod starts, enabling you to benefit from the lifecycle management capabilities of Secrets Manager. So, you not only gain the benefit of a natively-integrated secrets management solution, but also the ability to provide configurations in a single provider.


## **Overview**
In this post, I will show you how to set up AWS Secrets & Configuration Provider (ASCP) to work with the Secrets Store CSI driver on your Kubernetes clusters. The Secrets Store CSI driver allows Kubernetes to mount secrets stored in external secrets stores into the pods as volumes. After the volumes are attached, the data is mounted into the container’s file system. In this example, the external secret store is Secrets Manager.


![](Aspose.Words.9e4e6f6e-a418-4d87-bc4a-a7d131024ad6.001.png)



This solution includes the following steps, which will be described in more detail in the following sections:

1. Restrict access to your pods using IAM roles for service accounts
1. Install the Kubernetes secrets store CSI driver
1. Install the AWS Secrets & Configuration Provider
1. Create and deploy the SecretProviderClass custom resource
1. Configure and deploy the Pods to mount the volumes based on the configured secrets
1. Load secrets and configurations from the volumes mounted to the container


## **Prerequisites**
This solution has the following prerequisites:

- An AWS account.
- An IAM policy with permissions to retrieve a secret from Secrets Manager.
- Your secret stored in Secrets Manager
- An existing EKS Cluster
- A user that can modify your Kubernetes cluster
- AWS CLI and kubectl installed
- Helm and eksctl installed


## **Deploying the solution**
### **Step 1: Restrict access to your pods using IAM roles for service accounts**
You will use IAM roles for service accounts (IRSA) to limit secret access to your pods. By setting this up, the provider will retrieve the pod identity and exchange this identity for an IAM role. ASCP will then assume the IAM role of the pod and only retrieve secrets from Secrets Manager that the pod is authorized to access. This prevents the container from accessing secrets that are intended for another container that belongs to another pod.

Run the following command to turn on Open ID Connect (OIDC). Remember to replace *<REGION>* and *<CLUSTERNAME>* with your own values.


eksctl utils associate-iam-oidc-provider --region=*<REGION>* --cluster=*<CLUSTERNAME>* --approve #Only run this once


To create your service account role, run the following command to associate the policy (from the Prerequisites section) with your service account. Replace *<NAMESPACE>*, *<CLUSTERNAME>*, *<IAM\_policy\_ARN>*, *<SERVICE\_ACCOUNT\_NAME>* with your own values.


eksctl create iamserviceaccount --name *<SERVICE\_ACCOUNT\_NAME>* --namespace *<NAMESPACE>* --cluster *<CLUSTERNAME>* --attach-policy-arn *<IAM\_policy\_ARN>* --approve --override-existing-serviceaccounts


### **Step 2: Install the Kubernetes secrets store CSI driver**
From your terminal where you have kubectl installed, run the following helm commands to install the CSI driver.


helm repo add secrets-store-csi-driver[ ](https://raw.githubusercontent.com/kubernetes-sigs/secrets-store-csi-driver/master/charts)<https://raw.githubusercontent.com/kubernetes-sigs/secrets-store-csi-driver/master/charts>




Next, you need to determine whether you want to turn on automated rotation for the driver using the rotation reconciler feature, or whether you do not need to periodically pull updated secrets.

If you do not need to periodically pull updated secrets, initialize the driver with the following command:


helm -n kube-system install csi-secrets-store secrets-store-csi-driver/secrets-store-csi-driver 



Note: If running an older version of the driver, the flag –set grpcSupportedProviders=”aws” might be required.

If you want to turn on automated rotation for the driver using the rotation reconciler feature which is currently in alpha, use the command as follows (you can adjust the rotation interval as you desire to find an appropriate balance between API call cost consideration and rotation frequency):


helm -n kube-system install csi-secrets-store secrets-store-csi-driver/secrets-store-csi-driver --set enableSecretRotation=true --set rotationPollInterval=3600s




To validate that the installer is running as expected, run the following command:


kubectl get po --namespace=kube-system 



The output should show the following Secrets Store CSI driver pods and custom resource definitions (CRDs) deployed:

csi-secrets-store-qp9r8         3/3     Running   0          4m

csi-secrets-store-zrjt2         3/3     Running   0          4m

kubectl get crd

NAME                                               

secretproviderclasses.secrets-store.csi.x-k8s.io

Secretproviderclasspodstatuses.secrets-store.csi.x-k8s.io


### **Step 3: Install the AWS Secrets & Configuration Provider**
The CSI driver allows you to mount your secrets in your EKS Kubernetes pods. To retrieve them from Secrets Manager so the CSI driver can mount them, you need to install the AWS Secrets & Configuration Provider (ASCP). You do this by running the following command in your terminal, which will pull down the installer file without the need to clone the entire repo.

curl -s https://github.com/aws/secrets-store-csi-driver-provider-aws/blob/main/deployment/aws-provider-installer.yaml | kubectl apply -f - 


### **Step 4: Create and deploy the SecretProviderClass custom resource**
In order to use the Secrets Store CSI driver, you have to create a SecretProviderClass custom resource. This provides driver configurations and provider-specific parameters to the CSI driver itself. The SecretProviderClass resource should have at least the following components:

apiVersion: secrets-store.csi.x-k8s.io/v1alpha1

kind: SecretProviderClass

metadata:

`  `name: aws-secrets

spec:

`  `provider: aws                               

`  `parameters:                                 # provider-specific parameters


To use ASCP, you create the SecretProviderClass to provide a few more details of how you are going to retrieve secrets from Secrets Manager. The SecretProviderClass *MUST* be in the same namespace as the pod referencing it. The following is an example SecretProviderClass configuration:

apiVersion: secrets-store.csi.x-k8s.io/v1alpha1

kind: SecretProviderClass

metadata:

`  `name: aws-secrets

spec:

`  `provider: aws

`  `parameters:                    # provider-specific parameters

`    `objects:  |

`      `- objectName: "MySecret2"

`        `objectType: "secretsmanager"


### **Step 5: Configure and deploy the pods to mount the volumes based on the configured secrets**
Update your deployment YAML to use the secrets-store.csi.k8s.io driver, and reference the SecretProviderClass resource created previously. This should be saved on your local desktop.

The following is an example of how to configure a pod to mount a volume based on the SecretProviderClass to retrieve secrets from Secrets Manager. In this example, I used NGINX. But for your secret, the mount point and SecretProviderClass configuration will be in the pod deployment specification file.


kind: Pod

apiVersion: v1

metadata:

`  `name: nginx-secrets-store-inline

spec:

`  `serviceAccountName: aws-node

`  `containers:

`  `- image: nginx

`    `name: nginx

`    `volumeMounts:

`    `- name: mysecret2

`      `mountPath: "/mnt/secrets-store"

`      `readOnly: true

`  `volumes:

`    `- name: mysecret2

`      `csi:

`        `driver: secrets-store.csi.k8s.io

`        `readOnly: true

`        `volumeAttributes:

`          `secretProviderClass: "aws-secrets"



On pod start and restart, the CSI driver will call the provider binary to retrieve the secret and configurations from Secrets Manager and Parameter Store, respectively. After successfully retrieving this information, the CSI driver will mount them to the container’s file system. You can validate that the volume is mounted properly after a restart by running the following command:

kubectl exec -it nginx-secrets-store-inline -- ls /mnt/secrets-store/



You should get the following response:

MySecret2

### **Step 6: Load secrets and configurations from the volumes mounted to the container.**
Both secrets and configurations will be fetched at pod initialization during the mount operation. This can add a small amount of latency when using the native Kubernetes secrets, but it is similar to the experience of retrieving secrets through a custom or third-party tool. After initialization, your pod will not be impacted. ASCP, along with the rotation reconciler component, will update the values in the mount path and in the Kubernetes secret. The workload pods will watch the file system to track changes and automatically pick up new credentials. In the case of environmental variables, you will need to restart your pods.
