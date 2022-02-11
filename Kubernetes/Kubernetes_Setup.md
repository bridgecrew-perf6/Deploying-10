# Setup Kubernetes (K8s) Cluster on AWS


1. Create Ubuntu EC2 instance
1. Install AWSCLI
   ```sh
    curl curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    sudo apt update
    sudo apt install unzip python
    unzip awscliv2.zip
    sudo ./aws/install
    ./aws/install -i /usr/local/aws-cli -b /usr/local/bin
    ```

1. Install kubectl on ubuntu instance
   ```sh
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin/kubectl
   ```

1. Install kops on ubuntu instance
   ```sh
    curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
    chmod +x kops-linux-amd64
    sudo mv kops-linux-amd64 /usr/local/bin/kops
    ```
1. Create an IAM user/role  with Route53, EC2, IAM, SQS, EventBridge, VPC and S3 full access

1. Attach IAM role to ubuntu instance
   ```sh
   # Note: If you create IAM user with programmatic access then provide Access keys. Otherwise region information is enough
   aws configure
     Default region name [None]: ca-central-1
    ```

1. Create a Route53 private hosted zone (you can create Public hosted zone if you have a domain)
   ```sh
   Routeh53 --> hosted zones --> created hosted zone  
   Domain Name: jessm.com
   Type: Private hosted zone for Amazon VPC
   ```

1. create an S3 bucket
   ```sh
    aws s3 mb s3://k8s.s3bucket.jessm.com
   ```
1. Expose environment variable:
   ```sh
    export KOPS_STATE_STORE=s3://k8s.s3bucket.jessm.com
   ```

1. Create sshkeys before creating cluster (Required)
   ```sh
    ssh-keygen
   ```

1. Create kubernetes cluster definitions on S3 bucket
   ```sh
   kops create cluster --cloud=aws --zones=ca-central-1b --name=k8s.s3bucket.jessm.com --dns-zone=jessm.com --dns private 
    ```
   Validate the cluster configuration created into the S3 bucket
   ```sh 
   kops get cluster
   ```
   
1. Expose
   ```sh
   export KOPS_STATE_STORE=s3://k8s.s3bucket.jessm.com
   ```

1. Update the cluster worker node size to machineType: t2.micro (Vi editor)
   ```sh 
   kops edit ig --name=k8s.s3bucket.jessm.com master-ca-central-1b
   
   kops edit ig --name=k8s.s3bucket.jessm.com nodes-ca-central-1b
   ```

1. Create kubernetes cluster
    ```sh
    kops update cluster --name k8s.s3bucket.jessm.com --yes 
    ```
    Validate the cluster configuration created into the S3 bucket and the new ec2 instances created

1. ERROR trying to login the master node
   ```sh
   ssh to the master: ssh -i ~/.ssh/id_rsa ubuntu@api.k8s.test.jessm.com
   ubuntu@api.k8s.s3bucket.jessm.com: Permission denied (publickey).
   
   .
   .
   .
   

1. Validate your cluster
     ```sh
      kops validate cluster
    ```

1. To list nodes
   ```sh
   kubectl get nodes
   ```

1. To delete cluster
    ```sh
     kops delete cluster --name=k8s.s3bucket.jessm.com --yes
    ```
   
#### Deploying Nginx pods on Kubernetes
1. Deploying Nginx Container
    ```sh
    kubectl create deploy sample-nginx --image=nginx --replicas=2 --port=80
    # kubectl deploy simple-devops-project --image=yankils/simple-devops-image --replicas=2 --port=8080
    kubectl get all
    kubectl get pod
   ```

1. Expose the deployment as service. This will create an ELB in front of those 2 containers and allow us to publicly access them.
   ```sh
   kubectl expose deployment sample-nginx --port=80 --type=LoadBalancer
   # kubectl expose deployment simple-devops-project --port=8080 --type=LoadBalancer
   kubectl get services -o wide
