# Create Amazon EKS cluster and node group in new VPC with AWS CloudFormation


## 1. Prepare Environment

Refer to [AWS Cloud9](https://github.com/t2yijaeho/Docker-with-AWS-Cloud9)


## 2. Create an Amazon EKS cluster and node group


### 1. Get an AWS CloudFormation stack template body

1. Validate YAML-formatted text file [Stack Template Body](https://github.com/t2yijaeho/Amazon-EKS-with-CloudFormation/blob/matia/Template/K8sCluster.yaml)

    ```console
     wget https://github.com/t2yijaeho/Amazon-EKS-with-CloudFormation/raw/matia/Template/K8sCluster.yaml
    ```


### 2. Set a Kubernetes version

1. Check available [Amazon EKS Kubernetes versions](https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html)
   
    ***Change `<K8s Version>` to your desired kubernetes version***
   
    ```console
     K8S_VERSION=<K8s Version>
     echo $K8S_VERSION
    ```
    
    ```console
    mspuser:~/environment $ K8S_VERSION="1.22"
    mspuser:~/environment $ echo $K8S_VERSION
    1.22
    ```


### 3. Get an AMI ID for Kubernetes worker node EC2

1. Refer to [EKS optimized AMI](https://docs.aws.amazon.com/eks/latest/userguide/eks-optimized-ami.html) according to Kubernetes versioin

    ```cli
    AMI_ID=$(aws ssm get-parameter \
      --name /aws/service/eks/optimized-ami/$K8S_VERSION/amazon-linux-2/recommended/image_id \
      --query "Parameter.Value" --output text)
    echo $AMI_ID
    ```    
    
    ```console
    mspuser:~/environment $ AMI_ID=$(aws ssm get-parameter \
    >   --name /aws/service/eks/optimized-ami/$K8S_VERSION/amazon-linux-2/recommended/image_id \
    >   --query "Parameter.Value" --output text)
    mspuser:~/environment $ echo $AMI_ID
    ami-0123f8d52a516f19d
    mspuser:~/environment $ 
    ```


### 4. Create an AWS CloudFormation stack

1. Create stack `K8sBasics` using worker node AMI ID and Kubernetes version parameters

    ```console
    aws cloudformation create-stack \
      --stack-name K8sBasics \
      --template-body file://./K8sCluster.yaml \
      --parameters ParameterKey=WorkerImageID,ParameterValue=$AMI_ID \
                   ParameterKey=K8sVersion,ParameterValue=$K8S_VERSION \
      --capabilities CAPABILITY_IAM
    ```


### 5. Monitor the progress by the CloudFormation stack's events in AWS management console
   ***Launching will take approximately 15 minutes***

<img src="https://github.com/t2yijaeho/Amazon-EKS-with-CloudFormation/blob/matia/Images/CloudFormation%20Stack%20Creation%20Events.png?raw=true">

