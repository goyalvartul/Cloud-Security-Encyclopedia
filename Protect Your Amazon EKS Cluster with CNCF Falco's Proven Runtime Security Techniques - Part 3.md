# Protect Your Amazon EKS Cluster with CNCF Falco's Proven Runtime Security Techniques - Part 3


### **Set up the Falco Helm repository**
To use the falcosecurity/falco Helm chart, first clone the repository and then add the Helm chart.

git clone https://github.com/falcosecurity/charts.git; helm repo add falcosecurity https://falcosecurity.github.io/charts

### Helm repo update

Go to the falco/rules directory and examine the default configuration files for rules that are provided and ready to use. Review the default rule-set yaml files, which contain detailed explanations of the rules specified in each one. Custom rules can also be added.

The following files are included in the default rule configuration:

1. application\_rules.yaml
2. falco\_rules.local.yaml
3. falco\_rules.yaml
4. K8s\_audit\_rules.yaml

Falco's behavior can be controlled by configuration parameters, which can be supplied as runtime parameters during the installation of the chart or by creating a special purpose file, such as values.yaml (you can give any name).

Check out this link[ ](https://github.com/falcosecurity/charts/tree/master/falco#introduction)<https://github.com/falcosecurity/charts/tree/master/falco#introduction> to understand all the configuration parameters that control the run time behavior of Falco, such as audit level, log level, file outputs, etc.

A sample values.yaml file is provided for reference.




The jsonOutput property in the values.yaml file is set to false by default. To enable json formatted output through fluent-bit, set this property to true. You can find this file in the falcosecurity/charts repository on GitHub. 

To install the Helm chart for Falco, run the command "helm install falco -f values.yaml falcosecurity/falco" and the output should be as follows:

    NAME: falco
    LAST DEPLOYED: Tue Jan 6 12:06:26 2022
    NAMESPACE: default
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:

When you deploy Falco, it will automatically start running on each node in your cluster. After a short amount of time, it will begin monitoring your containers for any security issues. No additional steps are necessary. Once the deployment is finished, Falco will continuously scan your Kubernetes cluster pods for any security-related or abnormal activity and send the log events to Firelens. Firelens will then reformat the JSON logs based on the specified configuration settings and send them to CloudWatch. In the end, you will have a specific number of pods and deployments.

    kubectl get pods —all-namespaces

You will get the following output:

    NAMESPACE     NAME                      READY STATUS RESTARTS AGE
    
    default       falco-f95fw               1/1   Running 0       10d
    
    default       falco-n7hb8               1/1   Running 0       10d
    
    default       fluentbit-85bsm           1/1   Running 0       10d
    
    default       fluentbit-qjk5n           1/1   Running 0       10d
    
    default       nginx2-7844999d9c-8xgkx   1/1   Running 0       10d
    
    default       nginx2-7844999d9c-fbsqf   1/1   Running 0       10d
    
    default       nginx2-7844999d9c-wdpz8   1/1   Running 0       10d
    
    kube-system   aws-node-86mwx            1/1   Running 0       10d
    
    kube-system   aws-node-qwfs4            1/1   Running 0       10d
    
    kube-system   coredns-56666f95ff-dg75w  1/1   Running 0       10d
    
    kube-system   coredns-56666f95ff-xr8l2  1/1   Running 0       10d
    
    kube-system   kube-proxy-b54kq          1/1   Running 0       10d
    
    kube-system   kube-proxy-kp5wd          1/1   Running 0       10d
    
    In the CloudWatch Console, navigate to the Log group streams.



![](images/image24.png)

### **Simulating and Testing**
Falco is designed to detect any suspicious activity on the pods in your cluster. To test this, you can go inside one of the test deployments (in this case, the NGINX application pods) and run commands that will trigger the default rules set up in Falco. For example, you can use the command "kubectl get pods" to list all the running pods on your EKS cluster and then go inside one of the NGINX pods (e.g. "nginx2-7844999d9c-wdpz8") and perform actions that will trigger the rules specified in falco\_rules.yaml, such as "write below etc" and "Read sensitive file untrusted". This way Falco will detect the suspicious activities based on the rule sets we have created. Falco is enabled to catch all suspicious activities on all the pods on the whole cluster instances.

    kubectl exec -it 'nginx2-7844999d9c-wdpz8' /bin/bash

and produce the activity below:

    touch /etc/2

    cat /etc/shadow > /dev/null 2>&1

Falco will generate the alerts on CloudWatch as shown below on the test simulation.

![](images/image25.png)

![](images/image26.png)

Example 2: Go inside any of the Nginx pod and execute below statements, which will trigger the rule called “Mkdir binary dirs.”

    kubectl exec -it 'nginx2-7844999d9c-wdpz8' /bin/bash

and generate the activity like below:

    cd /bin

mkdir hello

As a result of the test simulation, Falco will produce alerts on CloudWatch, as demonstrated below.

![](images/image27.png)

### **Creating Custom Rules**
We are going to create a YAML file containing some custom rules, or add new rules to the existing default rule set. For this demonstration, I will create a new file called custom\_alerts.yaml, which will include the desired rules conditions. For example, I will create an alert for simple commands such as "whoami" and "who". When these commands are run inside the NGINX container, Falco will send an alert. Below is a sample custom\_alerts.yaml file:

customRules:

    rules-nginx.yaml: |
    \- macro: nginx\_consider\_syscalls
    condition: (evt.num < 0)
    \- macro: app\_nginx
    condition: container and container.image contains "nginx"
    \# Any outbound traffic raises a WARNING
    \- rule: The program "whoami" is run in a container
    desc: An event will trigger every time you run "whoami" in a container
    condition: evt.type = execve and evt.dir=< and container.id != host and proc.name = whoami
    output: "whoami command run in container (user=%user.name %container.info parent=%proc.pname cmdline=%proc.cmdline)"
    priority: NOTICE
    warn\_evttypes: False
    \- rule: The program "locate" is run in a container
    desc: An event will trigger every time you run "locate" in a container
    condition: evt.type = execve and evt.dir=< and container.id != host and proc.name = locate
    output: "locate command run in container (user=%user.name %container.info parent=%proc.pname cmdline=%proc.cmdline)"
    priority: NOTICE
    warn\_evttypes: False

To add the new custom rules to the security alerting, upgrade the Helm chart using the new configuration file. To do this, run the command "helm upgrade falco -f custom\_alerts.yaml falcosecurity/falco" and you should see the following output:

    Release "falcosec" has been upgraded. Happy Helming!

    NAME: falcosec
    LAST DEPLOYED: Tue Oct 6 12:11:47 2020
    NAMESPACE: default
    STATUS: deployed
    REVISION: 2
    TEST SUITE: None
    NOTES:

When you deploy Falco, it will automatically begin running on each node in your cluster. After a short period of time, it will start monitoring your containers for any security issues. No additional steps are necessary. Once the deployment is finished, you can SSH into one of the NGINX pods and execute specific commands that will trigger the custom rules you have set up.

    kubectl exec -it 'nginx2-7844999d9c-wdpz8' /bin/bash

and produce the activity below:

    whoami

    find

As a result of the test simulation, Falco will produce the following alerts on CloudWatch.


![](images/image28.png)

Similarly, you can create multiple custom rules based on the needs of your application or system.