# Protect Your Amazon EKS Cluster with CNCF Falco's Proven Runtime Security Techniques - Part 2


### **Set up a sample deployment on Amazon EKS Cluster**
Create a new configuration called deployment.yaml for your sample application. We will use a sample Nginx website pods on the public subnets that we have provided in the cluster configuration file cluster-config.yaml.

Check the below sample deployment.yaml file.

    apiVersion: apps/v1
    kind: Deployment
    metadata:
    `  `name: nginx2
    `  `labels:
    `    `app: nginx2
    spec:
    `  `replicas: 3
    `  `selector:
    `    `matchLabels:
    `      `app: nginx2
    `  `template:
    `    `metadata:
    `      `labels:
    `        `app: nginx2
    `    `spec:
    `      `affinity:
    `        `nodeAffinity:
    `          `requiredDuringSchedulingIgnoredDuringExecution:
    `            `nodeSelectorTerms:
    `            `- matchExpressions:
    `              `- key: beta.kubernetes.io/arch
    `                `operator: In
    `                `values:
    `                `- amd64
    `                `- arm64
    `      `containers:
    `      `- name: nginx
    `        `image: nginx:1.19.2
    `        `ports:
    `        `- containerPort: 80


Now deploy Nginx as shown below:

    kubectl apply -f deployment.yaml

You can verify the deployment status with kubectl command.

    kubectl get deployments --all-namespaces

You should see the following output.

    NAMESPACE     NAME      READY     UP-TO-DATE        AVAILABLE        AGE
    
    default                 nginx2       3/3                 3                           3                    46d
     
    kube-system       coredns      2/2                2                            2                   46d



### **Set up a Falco runtime security**

We will install the popular runtime security tool CNCF Falco for deep container security events analysis and alerts. Falco works in conjunction with other AWS services such as Firelens and Amazon CloudWatch. Firelens is a log aggregator product that can collect and send container logs to various services within the Amazon ecosystem, such as Amazon CloudWatch, for further analysis and alerting mechanisms. Firelens uses Fluent Bit or Fluentd behind the scenes and supports all features and configurations of both of the products. Additionally, AWS Firelens logs output can be sent to external logging and analytics services as well.

Amazon CloudWatch is a powerful monitoring, alerting and analytics service provided by Amazon that offers a wide range of insights on the services from which logs are received. It allows for custom dashboard metrics, alerting and insights on the logs. Please check the documentation here on Amazon CloudWatch. Falco specifically uses Firelens and Amazon CloudWatch as follows, as explained in the blog:

1. Falco continuously scans the containers running in the pods and sends the security, debug, or audit events as JSON format as STDOUT.
2. Firelens then collects the JSON log file and processes the logs files as per Fluent Bit configuration files.
3. After log transformation by Fluent Bit containers, the logs are finally sent to AWS CloudWatch as the final destination.

This blog post explains in depth on how to install Falco and how it works with other AWS services.


### Clone the Falco repository

To use the Falco-AWS-Firelens integration, first clone the repository from the link[ ](https://github.com/sysdiglabs/falco-aws-firelens-integration)<https://github.com/sysdiglabs/falco-aws-firelens-integration>. Then, navigate to the eks/fluent-bit directory. Within that directory, you will find two subdirectories called "aws" and "kubernetes". The "aws" directory contains an IAM policy named "iam\_role\_policy.json" that needs to be attached to the worker node's role in an EKS cluster in order to allow Falco to send logs to Amazon CloudWatch. The "kubernetes" directory has three files - configmap.yaml, daemonset.yaml, and service-account.yaml - that are used to create a ConfigMap for Fluent Bit configuration, a Fluent Bit DaemonSet to run on all worker nodes, and a service account for RBAC cluster role authorization. All of these files should be applied at the same time. This Falco blog explains the process of installing standard Falco and attaching the IAM policy to the node instances for logging to Amazon CloudWatch.

### Set up the Falco with IAM permissions

    aws iam create-policy --policy-name EKS-CloudWatchLogs --policy-document file://./fluent-bit/aws/iam\_role\_policy.json

This creates a policy called EKS-CloudWatchLogs with privileges to send logs to Amazon CloudWatch.

    aws iam attach-role-policy --role-name <EKS-NODE-ROLE-NAME> --policy-arn `aws iam list-policies | jq -r '.[][] | select(.PolicyName == "EKS-CloudWatchLogs") | .Arn'`


Please be aware that "EKS-NODE-ROLE-NAME" refers to the role that is associated with the worker nodes in your EKS cluster. This role can be located by checking the existing roles. For instance, in the case where an EKS cluster has been set up, the role may be named "eksctl-eks-managed-cluster-nodegr-NodeInstanceRole-1T0251NJ7YV04" which is associated with the node.

    aws iam attach-role-policy --role-name eksctl-eks-managed-cluster-nodegr-NodeInstanceRole-1T0251NJ7YV04 --policy-arn `aws iam list-policies | jq -r '.[][] | select(.PolicyName == "EKS-CloudWatchLogs") | .Arn'`

To complete the process, apply the entire directory, including all of the configuration files (configmap.yaml, daemonset.yaml, and service-account.yaml) listed within it.

    kubectl apply -f eks/fluent-bit/kubernetes/
