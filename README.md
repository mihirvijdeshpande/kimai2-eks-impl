# kimai2-eks-impl
Kimai2 implementation on AWS EKS:

##Prerequisites

**Install and Setup the following tools Locally**
 - kubectl
 - eksctl
 - helm
 - docker
 - aws cli

**Setup AWS**

 - Create an IAM Role in AWS to allow access to EKS cluster
 - Create key-pair in IAM using below command
    ```aws ec2 create-key-pair --region us-west-2 --key-name myKeyPair```
 - Create EKS cluster using eksctl:
    ```eksctl create cluster \
        --name my-cluster \
        --region us-west-2 \
        --with-oidc \
        --ssh-access \
        --ssh-public-key <your-key> \
        --managed```
