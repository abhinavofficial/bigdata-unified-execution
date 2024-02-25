# bigdata-unified-execution

## Infra Set up

### Motivation

Currently, we operate our big data services on infrastructure like AWS, Snowflake, Kafka, MongoDB, Oracle, SQL Server, Mainframe system, etc. Each of these infrastructures has its complexities to maintain requiring several platform engineers and administrators. On top of that, there are many engineers required to manage the network and its bandwidth, its integration points, firewalls and connectivity, etc. It not only complicates the data architecture but also complicates the ops model across its services.

Due to the choice of individual operational system requirements, constraints with third-party services and design requirements, we may not be able to remove these infrastructure layers and components. To scale efficiently, we need to abstract these infrastructure components from the actual execution layer.

This is where Environment as Code makes the most sense. We can start small and move on from there.

### Problem to solve

1. Can we create an interface for an environment?
2. Is it possible to create a new environment at will as defined by an interface?

## Test environment setup

* Install and set up minikube. Please follow the link [setup](https://minikube.sigs.k8s.io/docs/start/) guide.

```txt
*😄  *minikube v1.32.0 on Ubuntu 22.04
✨  Using the docker driver based on existing profile
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2900MB) ...
🐳  Preparing Kubernetes v1.28.3 on Docker 24.0.7 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🔎  Verifying Kubernetes components...
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

* Check if minikube setup is fine

```shell
minikube kubectl -- get po -A
alias kubectl="minikube kubectl --"
minikube dashboard

# verify before progress if you have sufficient privilege
kubectl auth can-i <list|create|edit|delete> <pods|services|configmaps>

minikube addons enable metrics-server
```

* Test if you can run your first spark job

```shell
$ kubectl create serviceaccount spark
$ kubectl create clusterrolebinding spark-role --clusterrole=edit --serviceaccount=default:spark --namespace=default

# to get the kubernetes control plane url. This return the secured (https) url
$ kubectl cluster-info

# Alternate, you can start local proxy
$ kubectl proxy

$ ./bin/spark-submit \
    --master k8s://http://127.0.0.1:8001 \
    --deploy-mode cluster \
    --name spark-pi \
    --class org.apache.spark.examples.SparkPi \
    --conf spark.executor.instances=5 \
    --conf spark.kubernetes.container.image=apache/spark \
    --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \ local:///opt/spark/examples/jars/spark-examples_2.12-3.5.0.jar
```

* Let's add the next level of complexity. In the local environment, we will hold out code and data files within S3 compatible minio.
  
* Now we keep the metadata in postgres. So, let's set that up as well.


## Production Environment set up

### AWS Environment

* Get AWS Account
* Install and confirm AWS CLI (configured with an admin profile) Note that your profile name will be "default" if you configure with aws configure without using a --profile argument. This will be important later.
* terraform
* kops
* kubectl if not provided by homebrew in the previous step

[Old Source](https://github.com/lynnlangit/Spark-Scala-EKS/blob/master/AWS-Setup-Guide-Spark-EKS.md)

[New Source](https://docs.aws.amazon.com/eks/latest/userguide/setting-up.html)

When first installing kubectl, it isn't yet configured to communicate with any server. We will cover this configuration as needed in other procedures. If you ever need to update the configuration to communicate with a particular cluster, you can run the following command. Replace region-code with the AWS Region that your cluster is in. Replace my-cluster with the name of your cluster.

aws eks update-kubeconfig --region region-code --name my-cluster

#### EMR on EKS Set up

https://aws.amazon.com/blogs/big-data/amazon-emr-on-eks-widens-the-performance-gap-run-apache-spark-workloads-5-37-times-faster-and-at-4-3-times-lower-cost/

https://aws.amazon.com/blogs/containers/best-practices-for-running-spark-on-amazon-eks/

https://aws.amazon.com/blogs/big-data/use-karpenter-to-speed-up-amazon-emr-on-eks-autoscaling/

https://aws.amazon.com/blogs/containers/run-spark-rapids-ml-workloads-with-gpus-on-amazon-emr-on-eks/

https://docs.aws.amazon.com/emr/latest/EMR-on-EKS-DevelopmentGuide/setting-up.html

### Snowflake Environment

### Kafka Environment

### MongoDB Environment

### Oracle Environment

### SQL Server Environment

### Postgres Environment

### Neo4j Environment

### Teradata Environment

## Engineering Plane Set up

## Data Plane Set up

## Control Plane Set up
