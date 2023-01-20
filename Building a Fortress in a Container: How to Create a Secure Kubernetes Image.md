# Building a Fortress in a Container: How to Create a Secure Kubernetes Image


Security is a major concern when it comes to containerization, particularly with regard to container images. A recent report by Red Hat on the State of Kubernetes and Container Security revealed that:

- 24% of significant container security issues are vulnerabilities that can be fixed
- 70% are misconfigurations
- 27% are runtime security incidents

These numbers can be daunting for organizations considering moving to containerized workloads. But as an engineer, you can play a key role in addressing these concerns and ensuring the security of container images. This post will provide information on the importance of securing container images and ways to begin implementing security measures.



## **Why Check Container Images**

It is common practice for engineers to trust container images, such as Docker images, without thoroughly examining their contents. For example, once a Docker image is built, it is not typically opened and inspected for its directory structure or the base image directories used in its construction. Engineers often rely on trust in images found on Docker Hub without considering the security implications.

It is important to remember that while trust is necessary, it is also important to confirm the technical components of an image. This is known as the defense in depth principle, which states that multiple layers of security measures should be implemented in order to ensure safety. As engineers, it is our responsibility to regularly run security checks on images, such as the ubuntu:latest Docker image, and understand how they were built and what security practices were used in their construction.


When it comes to container security, it is important to avoid:

- Being held responsible for committing an image with security vulnerabilities that are later discovered
- Causing a complete shutdown of an organization due to a security vulnerability in a base image or any container image used throughout the application stack

Although it is not possible to completely eliminate all security risks, there are several effective measures that can be taken to minimize them. It is important to take proactive steps to mitigate as many security risks as possible.

## **Docker Commands For Security Checks**

The Docker CLI includes a useful security tool called "docker image inspect." Although it was not specifically designed for security purposes, it provides a wealth of information about a Docker image, including the container image hash. This hash can be used to detect if any changes have been made to the image and to verify that the image is coming from a trusted source. The command below shows how to pull an image called adminturneddevops/gowebapi from Docker Hub and the sha256 digest.

Example:

    docker image inspect adminturneddevops/gowebapi


![](images/image7.png)



When you use the command "docker inspect adminturneddevops/gowebapi:latest" on the container image, you will see that the sha256 hash is the same.


![](images/image8.png)

To ensure that the container image originates from the correct source, such as Docker Hub in this example, you can check that the sha256 hash matches the one listed on Docker Hub.

![](images/image9.png)


This allows you to confirm that you’re using the container image that you were expecting.
## **Snyk For Securing Docker Images**
The Docker CLI also includes the command "docker scan," which is powered by Snyk. Snyk is a tool that checks for vulnerabilities in container images based on a set of best practices. Docker and Snyk have partnered to integrate security directly into any containerized workload. To use Snyk, you can run the command:

docker scan containerimage:containerversion


For instance, to scan the "ubuntu:latest" container image, you would use the command:

docker scan ubuntu:latest



![](images/image10.png)

When using the "docker scan" command, the output will show a detailed list of any vulnerabilities found. This includes a summary of the vulnerabilities discovered, information on what was tested and on which platform it was run.

![](images/image11.png)

Vulnerabilities can be classified as either basic or crucial. As an illustration, the image below is a screenshot of a scan on the nginx:latest image on Docker Hub. The scan revealed a vulnerability to a SQL injection attack, which is a frequently occurring type of web attack.




![](images/image12.png)

To summarize, in a highly secure production environment, it is important to ensure that the developer tools being used meet your standards. Container images, which are used to deploy applications, are often built by individuals other than yourself, as they are often based on a base image. Thus, it is a good practice to verify the security of the container image before using it.



