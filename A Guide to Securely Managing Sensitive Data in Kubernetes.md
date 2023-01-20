# A Guide to Securely Managing Sensitive Data in Kubernetes

Secrets in Kubernetes are a secure means of storing sensitive information, including passwords, OAuth tokens, and SSH keys, among other data. These objects are encrypted and can be used within clusters. Some key features of Secrets include:

1. They are namespaced objects, providing a way to organize and manage them.
1. They can be mounted as data volumes or environment variables, making them easily accessible to containers in pods.
1. Secret data is stored in tmpfs on nodes, adding an additional layer of security.
1. The API server stores secrets as plain text in etcd, which allows for easy access and management.
1. There is a limit of 1MB per secret, ensuring efficient use of resources.

Creating a secret:

Create username.txt and password.txt files.


    echo -n 'root' > ./username.txt

    echo -n 'Mq2D#(8gf09' > ./password.txt

And

    kubectl create secret generic db-cerds \
    ` `--from-file=./username.txt \
    ` `--from-file=./password.txt

    secret "db-cerds" created

List secret:

    kubectl get secret/db-cerds
    NAME       TYPE      DATA      AGE
    db-cerds   Opaque    2         26s



View secret:

    kubectl describe secret/db-cerds
    Name:         db-cerds
    Namespace:    default
    Labels:
    Annotations:
    Type:  Opaque
    Data

\====

    password.txt:  11 bytes
    username.txt:  4 bytes


Using YAML file:

Secrets consist of two maps, "data" and "string data", where the "data" map is utilized to store any type of data in an encoded format using base64.


    echo -n 'root' | base64
    cm9vdA==
    echo -n 'Mq2D#(8gf09' | base64
    TXEyRCMoOGdmMDk=


Create a Secret yaml file

\---

    apiVersion: v1
    data:
    ` `password: TXEyRCMoOGdmMDk=
    ` `username: cm9vdA==
    kind: Secret
    metadata:
    ` `name: database-creds
    type: Opaque


Create the secret using kubectl create

    kubectl create -f creds.yaml
    secret "database-creds" created
    kubectl get secret/database-creds
    NAME             TYPE      DATA      AGE
    database-creds   Opaque    2         1m

View secret:

    kubectl get secret/database-creds -o yaml**

\---

    apiVersion: v1
    data:
    ` `password: TXEyRCMoOGdmMDk=
    ` `username: cm9vdA==
    kind: Secret
    metadata:
    ` `creationTimestamp: 2019-02-25 06:22:37 +00:00
    ` `name: database-creds
    ` `namespace: default
    ` `resourceVersion: "2657"
    ` `selfLink: /api/v1/namespaces/default/secrets/database-creds
    ` `uid: bf0cef90-38c5-11e9-8c95-42010a800068
    type: Opaque

Decoding secret values:

    echo -n "cm9vdA==" | base64 --decode
    root
    echo -n "TXEyRCMoOGdmMDk=" | base64 --decode
    Mq2D#(8gf09


Secrets can be employed in the following ways to support your workloads:

1. Setting environment variables that reference the values within the Secret
1. Mounting a volume that contains the Secret.

**Environment variables:**


\---

    apiVersion: v1
    kind: Pod
    metadata:
    ` `name: php-mysql-app
    spec:
    ` `containers:`   `
    `     `env:
    `         `name: MYSQL\_USER
    `         `valueFrom:
    `           `secretKeyRef:
    `             `key: username
    `             `name: database-creds
    `         `name: MYSQL\_PASSWORD
    `         `valueFrom:
    `           `secretKeyRef:
    `             `key: password
    `             `name: database-creds
    `     `image: "php:latest"
    `     `name: php-app



Secret as Volume:


\---

    apiVersion: v1
    kind: Pod
    metadata:
    ` `name: redis-pod
    spec:
    ` `containers:
    `     `image: redis
    `     `name: redis-pod
    `     `volumeMounts:
    `         `mountPath: /etc/dbcreds
    `         `name: dbcreds
    `         `readOnly: true
    ` `volumes:
    `     `name: dbcreds
    `     `secret:
    `       `secretName: database-creds

Additional Info :

Secret creation syntax

    kubectl create secret [TYPE] [NAME] [DATA]

TYPE refers to the type of Secret that is being created and it can be one of the following options:

- generic: This option allows you to create a Secret from a local file, directory or literal value.
- docker-registry: This option creates a dockercfg Secret for use with a Docker registry, which is used for authentication against Docker registries.
- tls: This option creates a TLS secret from an existing public/private key pair. The public key certificate must be PEM encoded and match the private key.

DATA refers to the type of data that is being used to create the Secret and it can include one of the following options:

    — from-file
    kubectl create secret generic credentials \ --from-file=username=./username.txt \ --from-file=password=./password.txt
    --from-env-file
    — from-env-file

    cat credentials.txt
    
    username=admin
    password=Ex67Hn\*9#(jw

    kubectl create secret generic credentials \
    --from-env-file ./credentials.txt
    
    — from-literal flags
    
    kubectl create secret generic literal-token \
    --from-literal user=admin \
    --from-literal password="Ex67Hn\*9#(jw"
