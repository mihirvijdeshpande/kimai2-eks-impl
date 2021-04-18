# kimai2 Implementation on AWS EKS
##Dummy Change
## Deployment Design

 - The Kimai2 multicontainer application will be deployed on AWS EKS.<br>
 - The decision to deploy the whole stack on EKS is taken based on the fact that we would have only one control point, kubernetes, thus making it easier and faster to manage both, the application as well as its database.<br>
 - The other solution explored included deploying App containers on ECS and leveraging AWS RDS for database<br>
 - Helm charts are also readily available for Kimai2 Kubernetes deployment and hence make deployment and maintenance easier.

**Architecture**

![Kimai Architecture](/assets/images/archi.png)

 - The Idea is to expose the Kimai2 application via an Ingress.<br>
 - Two deployments, one each for the application and DB, with services of their own to interact with each other can be craeted.<br>
 - Persistent Volumes to be used to store data that we want to persist in the event of failure.<br>
 - Replicasets to be set to 2 for Kimai Application in case of DR<br>
 - Current Design uses only on DB pod. StatefulSets can be used in case multiple DB pods are required.

## Prerequisites

**Install and Setup the following tools Locally**
 - [kubectl](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)
 - [eksctl](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)
 - [helm](https://helm.sh/docs/intro/install/)
 - [docker](https://docs.docker.com/engine/install/)
 - [aws cli](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html)

**Setup AWS**

 - Create an IAM Role in AWS to allow access to EKS cluster
 - Create key-pair in IAM using command: 
    ```
    aws ec2 create-key-pair --region us-west-2 --key-name myKeyPair
    ```
 - Create EKS cluster using eksctl:
    ```
        eksctl create cluster \
        --name my-cluster \
        --region us-west-2 \
        --with-oidc \
        --ssh-access \
        --ssh-public-key <your-key> \
        --managed
        ```
 - Create node-group on EKS:
    ```
        eksctl create nodegroup \
        --cluster <my-cluster> \
        --region <us-west-2> \
        --name <my-mng> \
        --node-type <m5.large> \
        --nodes <3> \
        --nodes-min <2> \
        --nodes-max <4> \
        --ssh-access \
        --ssh-public-key <my-key> \
        --managed
    ```
    OR
    Create using Launch Template file `eks-nodes.yaml` with command: 
    ```
        eksctl create nodegroup --config-file eks-nodegroup.yaml
    ```
**Configuring kubectl to work with EKS**

 - Configure aws cli using command: `aws configure`
 - Configure kubectl using command: `aws eks --region <region-code> update-kubeconfig --name <cluster_name>`
 - Test Configurations : `kubectl get svc`

## Install the Helm Chart for kimai2

 - Add helm repository: `helm repo add kimai2tet https://gresci.github.io/kimai2-helmchart/`
 - Install Helm chart for kimai2 on EKS cluster: `helm install my-kimai-helmchart kimai2tet/kimai-helmchart --version 0.1.0`

**Helm Values used in the deployed chart**

The values deployed in the chart are default taken from the values.yaml file in the [official repository](https://github.com/tobybatch/kimai2/tree/main/docs/helm) for kimai2 helm chart.
To change these values, we can clone the code, change values as needed and then install the helm chart on EKS cluster

## Expected Results

The deployment consists of following Kubernetes components:

 - 2 Deployments:
    - kimai deployment contains 1 replicaset managing kimai application pods
    - mysql deplyoment contains 1 replicaset managing mysql application pods
    - Both Deployments attach a Persistant Volume to the Pods to store non-ephemeral data
 - 2 Services each for Kimai app and MySQL
 - 1 Ingress to route traffic to Kimai application.
 - 1 Secret

![Kimai Setup](/assets/images/kimai-all.png)

## Pre-applied settings

As is much evident from the `values.yaml` file in the helm installation, most of the configurations needed are already set. No additional settings should be necessary.<br>
The DB Connector is set in the environment of the Kimai Docker container as `DATABASE_URL`<br>


## Creating Dummy Data to test

 - run `kubectl exec --stdin --tty <pod-name> -- /bin/bash` to get a shell in the pod
 - run command `/opt/kimai/bin/console kimai:create-user admin admin@example.com ROLE_SUPER_ADMIN password` to create an admin user for Kimai
 - Then import test data using this command: `opt/kimai/bin/console kimai:reset-dev`


