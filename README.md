# Kubernetes Basics with AWS Cloud9


## 1. Create an AWS Cloud9 Environment

   [AWS Cloud9](https://github.com/t2yijaeho/Docker-with-AWS-Cloud9)


## 2. Create an Amazon EKS cluster


1. Get an AWS CloudFormation stack template body

    ```console
    wget https://github.com/t2yijaeho/K8s-with-Cloud9/raw/matia/Template/K8sCluster.yaml
    ```

2. Get a AMI ID for Kubernetes worker node EC2

   [Kubernetes version](https://docs.aws.amazon.com/eks/latest/userguide/platform-versions.html)

   [EKS Optimized AMI](https://docs.aws.amazon.com/eks/latest/userguide/eks-optimized-ami.html)
   
   ***Change `<K8s Version>` to your desired kubernetes version***
   
    ```console
    K8S_VERSION=<K8s Version>
    echo $K8S_VERSION
    ```

    ```console
    AMI_ID=$(aws ssm get-parameter \
      --name /aws/service/eks/optimized-ami/$K8S_VERSION/amazon-linux-2/recommended/image_id \
      --query "Parameter.Value" --output text)
    echo $AMI_ID
    ```    

2. Create an AWS CloudFormation stack

    ```console
    aws cloudformation create-stack \
      --stack-name K8sBasics \
      --template-body file://./K8sCluster.yaml \
      --parameters ParameterKey=WorkerImageID,ParameterValue=$AMI_ID \
                   ParameterKey=K8sVersion,ParameterValue=$K8S_VERSION \
      --capabilities CAPABILITY_IAM
    ```

3. Monitor the progress by the CloudFormation stack's events in AWS management console (

    <img src="https://github.com/t2yijaeho/Docker-with-AWS-Cloud9/blob/matia/images/CloudFormation%20Stack%20Creation%20Events.png?raw=true">
    

## 3. Configure Kubernetes control plane

1. Install kubectl (Kubernetes command line utility)

    [AWS Guide to install kubectl](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)
    
    ***Set `kubectl version`, `release date` according to Kubernetes version***
    
    ```console
    KUBECTL_VERSION=<kubectl version>
    RELEASE_DATE=<release date>
    echo $KUBECTL_VERSION
    echo $RELEASE_DATE
    ```
    ```console
    mspuser:~/environment $ KUBECTL_VERSION=1.22.6
    mspuser:~/environment $ RELEASE_DATE=2022-03-09
    mspuser:~/environment $ echo $KUBECTL_VERSION
    1.22.6
    mspuser:~/environment $ echo $RELEASE_DATE
    2022-03-09
    mspuser:~/environment $ 
    ```
    
    Download the Amazon EKS vended kubectl binary
    
    ```console
    curl -o kubectl https://s3.us-west-2.amazonaws.com/amazon-eks/$KUBECTL_VERSION/$RELEASE_DATE/bin/linux/amd64/kubectl
    ```

    Apply execute permissions to the binary
    ```console
    chmod +x ./kubectl
    ```
    
    Copy the binary to a folder in PATH
    ```console
    mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
    ```
    
    Verify kubectl version
    ```console
    kubectl version --short --client
    ```

    ```console
    mspuser:~/environment $ kubectl version --short --client
    Client Version: v1.22.6-eks-7d68063
    mspuser:~/environment $ 
    ```
