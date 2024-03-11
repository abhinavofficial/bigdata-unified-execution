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

* Get spark version you want to run

```shell
wget https://dlcdn.apache.org/spark/spark-3.5.1/spark-3.5.1-bin-hadoop3.tgz
tar -xvf spark-3.5.1-bin-hadoop3.tgz
sudo mv spark-3.5.1-bin-hadoop3 /opt
sudo ln -s spark-3.5.1-bin-hadoop3/ spark
```

* Install and set up minikube. Please follow the link [setup](https://minikube.sigs.k8s.io/docs/start/) guide.

```shell
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube start

#  minikube v1.32.0 on Ubuntu 22.04
#  Using the docker driver based on existing profile
#  Starting control plane node minikube in cluster minikube
#  Pulling base image ...
#  Restarting existing docker container for "minikube" ...
#  Preparing Kubernetes v1.28.3 on Docker 24.0.7 ...
#  Configuring bridge CNI (Container Networking Interface) ...
#  Verifying Kubernetes components...
#    Using image gcr.io/k8s-minikube/storage-provisioner:v5
#    Using image gcr.io/k8s-minikube/minikube-ingress-dns:0.0.2
#  Enabled addons: storage-provisioner, ingress-dns, default-storageclass
#  kubectl not found. If you need it, try: 'minikube kubectl -- get pods -A'
#  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

* Check if minikube setup is fine

```shell
nikube kubectl -- get po -A
alias kubectl="minikube kubectl --"
```

* Spark expects DNS enabled Kubernetes

```shell
minikube addons enable ingress-dns
minikube start
```

* Set appropriate permissioning for running the spark application

```shell
# RBAC
kubectl create serviceaccount spark
kubectl create clusterrolebinding spark-role --clusterrole=edit --serviceaccount=default:spark --namespace=default

# verify before progress if you have sufficient privilege
kubectl auth can-i <list|create|edit|delete> <pods|services|configmaps>
```

* Get the cluster information you need to start the spark job

```shell
# to get the kubernetes control plane url. This return the secured (https) url
kubectl cluster-info
# Below is how the result looks
# > Kubernetes control plane is running at https://192.168.49.2:8443
# > CoreDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

# Alternate, you can start local proxy
kubectl proxy
```

* Test if you can run your first spark job

```shell
# Now do a spark submit to run the app
./bin/spark-submit --master k8s://https://192.168.49.2:8443 --deploy-mode cluster --name spark-pi --class org.apache.spark.examples.SparkPi --conf spark.executor.instances=1  --conf spark.kubernetes.container.image=apache/spark  --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark  local:///opt/spark/examples/jars/spark-examples_2.12-3.5.1.jar
```

* Now watch your applications runs

```shell
# To see things in dashboard
minikube dashboard

# Some dashboard features require the metrics-server addon. To enable all features please run:
minikube addons enable metrics-server
```

* There are some more things you can do here

```text
Create namespace and put resource limits or quotas per namespace. This helps manage blast radius.
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

[Blog 1](https://aws.amazon.com/blogs/big-data/amazon-emr-on-eks-widens-the-performance-gap-run-apache-spark-workloads-5-37-times-faster-and-at-4-3-times-lower-cost/)

[Blog 2](https://aws.amazon.com/blogs/containers/best-practices-for-running-spark-on-amazon-eks/)

[Blog 3](https://aws.amazon.com/blogs/big-data/use-karpenter-to-speed-up-amazon-emr-on-eks-autoscaling/)

[Blog 4](https://aws.amazon.com/blogs/containers/run-spark-rapids-ml-workloads-with-gpus-on-amazon-emr-on-eks/)

[Blog 5](https://docs.aws.amazon.com/emr/latest/EMR-on-EKS-DevelopmentGuide/setting-up.html)

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
