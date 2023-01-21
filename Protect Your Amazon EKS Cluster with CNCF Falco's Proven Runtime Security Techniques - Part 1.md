# Protect Your Amazon EKS Cluster with CNCF Falco's Proven Runtime Security Techniques - Part 1


A lot of companies are currently in the process of moving their applications to containers. Containers enable management of application-level dependencies, fast launches and support immutability. This can help decrease expenses, boost speed and enhance efficiency.

For ensuring the security of the container lifecycle, it is crucial to consider factors such as container image hardening and end-to-end security checks. Containers should be secured by default before they are deployed into a container orchestrator like Amazon Elastic Kubernetes Service (Amazon EKS). The process of hardening containers involves the following steps:

- Linting the Dockerfile
- Building the image with the lint-ed Dockerfile or Docker Compose file
- Running static container image scanning
- Examining the vulnerabilities
- Having a manual approval process
- Deploying to the orchestrator, Amazon ECS or Amazon EKS
- Enabling dynamic image scanning on Containers and regularly monitoring the logs

To better understand the pipeline flow, let's first go over what a static scan and a dynamic scan are:

- A static scan is a thorough examination of the container layers before they are used or deployed. The container is scanned against public bug or CVE databases.
- A dynamic scan is a thorough examination of the container layers after or while they are running or deployed. This method can scan and publish results as needed or continuously analyze the logs while the container is running. There are several products available in the market that fall under dynamic scanning tools, such as CNCF Falco, Twistlock, and Aqua.


In this article, we will demonstrate how to create, install, and use runtime security with CNCF Falco on Amazon EKS. The demo employs a static scan methodology, performs a thorough container scan for vulnerabilities or issues, and finally deploys them to Amazon EKS. We will be utilizing the following AWS services and open-source tools for this post:

- Amazon Elastic Kubernetes Service (Amazon EKS)
- Amazon CloudWatch
- Firelens
- AWS CloudFormation
- AWS CLI
- CNCF Falco
- falcosecurity/falco



### **Set up your Amazon EKS cluster**
Before setting up an Amazon EKS cluster, please install the following tools on your system. Detailed instructions on how to set them up for different operating systems can be found in the linked resources:

- ekstcl
- kubectl
- helm
- jq
- aws cli

Create a sample Amazon EKS cluster configuration file, called cluster-config.yaml. This configuration file will be used to deploy the Amazon EKS cluster. We can deploy the Amazon EKS cluster to an existing or new VPC. I have used an existing VPC to set up the cluster with managed node groups in both public and private subnets.

You can find many examples of ClusterConfig samples on the eksctl.io page. Below is one of the configuration files you can use. In this demo, we demonstrate how to build the cluster with pre-existing resources. Visit the eksctl docs to understand the ClusterConfig schema elements.



cluster-config.yaml file:

    apiVersion: eksctl.io/v1alpha5
    kind: ClusterConfig
    metadata:
    `  `name: eks-managed-cluster
    `  `region: ap-south-1
    vpc:
    `  `id: "vpc-xxxxxxxxxx" # Provide the VPC ID 
    `  `cidr: "xxxxxxxxxxxx" # Provide the VPC CIDR Range
    `  `subnets:
    `    `public:
    `      `ap-south-1a:
    `          `id: "subnet-xxxxxxxx" # Provide the Subnet ID
    `          `cidr: "xxxxxxxxxxxx" # Provide the Subnet CIDR Range
    `      `ap-south-1b:
    `          `id: "subnet-xxxxxxxx" # Provide the Subnet ID
    `          `cidr: "xxxxxxxxxxxx" # Provide the Subnet CIDR Range 

        

    \# Provide the service role for EKS cluster         
    #iam:
    \#  serviceRoleARN: "arn:aws:iam::11111:role/eks-base-service-role"
    \# Below schema elements build Non-EKS managed node groups
    #nodeGroups:
    \#  - name: ng-1
    \#    instanceType: m5.large
    \#    desiredCapacity: 3
    \#    iam:
    \#      instanceProfileARN: "arn:aws:iam::11111:instance-profile/eks-nodes-base-role"
    \#      instanceRoleARN: "arn:aws:iam::1111:role/eks-nodes-base-role"
    \#    privateNetworking: true
    \#    securityGroups:
    \#      withShared: true
    \#      withLocal: true
    \#      attachIDs: ['sg-xxxxxx', 'sg-xxxxxx']
    \#    ssh:
    \#      publicKeyName: 'my-instance-public-key'
    \#    tags:
    \#      'environment:basedomain': 'example.org'
    \# Below schema elements build EKS managed node groups
    managedNodeGroups:
    \- name: eks-managed-ng-1 # Provide the name of the node group
    minSize: 1 # Autoscaling Group configuration
    maxSize: 2 # Autoscaling Group configuration
    instanceType: t2.small # Size and type of the worker nodes
    desiredCapacity: 1 # Autoscaling Group configuration
    volumeSize: 20 # Worker Node volume size
    ssh:
       allow: true
    \# You can use the provided public key to logon to the containers.
    publicKeyPath: ~/.ssh/id\_rsa.pub
    \# sourceSecurityGroupIds: ["sg-xxxxxxxxxxx"]. # OPTIONAL
    labels: {role: worker}
    tags:
       nodegroup-role: worker
    iam:
    withAddonPolicies:
    externalDNS: true
    certManager: true
    \# provide the role ARN to be atatched to instances
    \#    iam:
    \#      instanceRoleARN: "arn:aws:iam::1111:role/eks-nodes-base-role"

Execute the following command to create the Amazon EKS cluster:

    eksctl create cluster -f cluster-config.yaml

You can locate the cluster created by Amazon CloudFormation as shown below:



![](images/image22.png)

You can check the status of the cluster creation on the Amazon EKS page of the AWS Management Console as follows:



![](images/image23.png)



