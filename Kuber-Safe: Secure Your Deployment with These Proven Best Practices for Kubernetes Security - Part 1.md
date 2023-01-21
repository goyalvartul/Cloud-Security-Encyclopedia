# Kuber-Safe: Secure Your Deployment with These Proven Best Practices for Kubernetes Security - Part 1

Kubernetes offers various controls that can significantly enhance the security of your application. However, properly configuring them requires a thorough understanding of Kubernetes and the security needs of the deployment. The best practices outlined here align with the container lifecycle, covering the stages of build, ship, and run, and are specifically designed for Kubernetes deployments.

Here are some recommendations for deploying a secure Kubernetes application:

**Ensure That Images Are Free of Vulnerabilities:**

Running containers with vulnerabilities leaves your environment vulnerable to being easily compromised. Many of these attacks can be prevented by ensuring that there are no software components with known vulnerabilities.

**Implement Continuous Security Vulnerability Scanning:**

Containers may contain outdated packages with known vulnerabilities (CVEs). This cannot be a one-time process, as new vulnerabilities are discovered regularly. A continuous process where images are constantly evaluated is crucial to maintain a required security posture.

**Regularly Apply Security Updates to Your Environment:**

Once vulnerabilities are found in running containers, the source image should always be updated and the containers should be redeployed. Avoid making direct updates (e.g. 'apt-update') to running containers, as this can disrupt the image-container relationship. Upgrading containers is made simple with Kubernetes' rolling updates feature - which allows for a gradual update of a running application by upgrading its images to the latest version.

**Ensure That Only Authorized Images are Used in Your Environment.**

Without a process in place to ensure that only images that comply with the organization's policy are allowed to run, the organization is at risk of running vulnerable or malicious containers. Downloading and running images from unknown sources is dangerous and similar to running software from an unknown vendor on a production server.

To mitigate this risk, use private registries to store approved images and ensure that only approved images are pushed to these registries. This alone will greatly reduce the number of potential images that enter your pipeline, as compared to the hundreds of thousands of publicly available images.

Implement a CI pipeline that incorporates security assessment, such as vulnerability scanning, as a part of the build process. The pipeline should ensure that only vetted code that is approved for production is used for building images. Once an image is built, it should be scanned for security vulnerabilities. If no issues are found, the image can then be pushed to a private registry from which deployment to production is done.

A failure in the security assessment should result in a failure in the pipeline, preventing images with poor security quality from being pushed to the image registry. There is ongoing work in Kubernetes for image authorization plugins, which will allow for preventing the shipping of unauthorized images.

**Limit Direct Access to Kubernetes Nodes**

To reduce the risk of unauthorized access to host resources, you should restrict SSH access to Kubernetes nodes. Instead, you should encourage users to use "kubectl exec" which will give direct access to the container environment without the ability to access the host.

You can also use Kubernetes authorization plugins to more precisely control user access to resources. This enables you to define precise access control rules for specific namespaces, containers, and operations.

**Create Administrative Boundaries between Resources**

Limiting user permissions can decrease the impact of errors or malicious activities. A Kubernetes namespace allows for partitioning created resources into logically named groups. Resources created in one namespace can be hidden from other namespaces.

By default, each resource created by a user in a Kubernetes cluster runs in the default namespace, called default. Additional namespaces can be created and resources and users can be assigned to them. Kubernetes Authorization plugins can be used to create policies that separate access to namespace resources between different users.

For example, the following policy will allow "john" to read pods from namespace 'alpha'.

    {
    `  `"apiVersion": "abac.authorization.kubernetes.io/v1beta1",
    `  `"kind": "Policy",
    `  `"spec": {
    `    `"user": "john",
    `    `"namespace": "alpha",
    `    `"resource": "pods",
    `    `"readonly": true
    `  `}
    }

