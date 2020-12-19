# Setup Kubernetes (K8s) Cluster on AWS


1. Create Ubuntu EC2 instance

1. Install AWSCLI
   ```sh
   #sudo apt-get update
   #curl https://s3.amazonaws.com/aws-cli/awscli-bundle.zip -o awscli-bundle.zip
   #sudo apt install unzip python
   #unzip awscli-bundle.zip
   #sudo ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
   
   ## Check result
#aws --version
   ## Change hostname of instance to k8s-management:
   #sudo hostname k8s-management
   ## Logout and login for the change affect
   ```
   
1. Install kubectl on ubuntu instance
   ```sh
   #curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
   #chmod +x ./kubectl
   #sudo mv ./kubectl /usr/local/bin/kubectl
   ```

1. Install kops on ubuntu instance
   ```sh
   #curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
   #chmod +x kops-linux-amd64
   #sudo mv kops-linux-amd64 /usr/local/bin/kops
   ```
   
1. Create an IAM user/role  with Route53, EC2, IAM and S3 full access

    On AWS console web page:

    ```
    Services > IAM link > IAM Home screen > Roles link > Create Role button
    Create Role screen > EC2 link > Next: Permissions > Attach permission policies
    - AmazonEC2FullAccess
    - AmazonS3FullAccess
    - AmazonRoute53FullAccess
    - IAMFullAccess
    Next: Tags > Next: Review > Create Role
    - Name=k8s-role
    ```

1. Attach IAM role to ubuntu instance
   
   On AWS console web page:
   
   ```
EC2 Dashboard > Instances list > choose Ubuntu k8s-management instance
   Action combobox > Security > Modify IAM role
   Modify AIM role screen > choose k8s-role
   ```
   
1. Change default Region for AWSCLI
   
   On k8s-management:
   
   Note: If you create IAM user with programmatic access then provide Access keys. Otherwise region information is enough

   ```sh
   #aws configure
   AWS Access Key ID [None]:
   AWS Secret Access Key [None]:
   Default region name [None]: us-east-2
   Default output format [None]:
   ```
   
1. Create a Route53 private hosted zone (you can create Public hosted zone if you have a domain)
   
   On AWS console web page:
   
   ```sh
   Route53 > Hosted zones --> Created hosted zone  
Domain Name: ipaperhub.com
   Type: Private hosted zone
   Region:us-east-2
   VPC: <choose VPC>
   ```
   
1. Create an S3 bucket
   ```sh
   #aws s3 mb s3://demo.k8s.ipaperhub.com
   ```
   
1. Expose environment variable:
   ```sh
   #export KOPS_STATE_STORE=s3://demo.k8s.ipaperhub.com
   ```

1. Create sshkeys before creating cluster
   ```sh
    #ssh-keygen
   ```

1. Create kubernetes cluster definitions on S3 bucket
    ```sh
    #kops create cluster --cloud=aws --zones=us-east-2a --name=demo.k8s.ipaperhub.com --dns-zone=ipaperhub.com --dns private 
    ```

    *Note: the `--zone` need the zonename with exactly suffix `a|b|c` (e.g.: us-east-2`a`). Refer the EC2 instance dashboard to know which a|b|c that instances are allocated.*

    List clusters with: 

    ```sh
    #kops get cluster
    ```

    Edit this cluster with: 

    ```sh
    #kops edit cluster demo.k8s.ipaperhub.com
    ```

    Edit node instance group: 

    ```sh
    #kops edit ig --name=demo.k8s.ipaperhub.com nodes
    ```

    *Note: change machineType: t3.medium to t2.micro for Free*

    Edit your master instance group: 

    ```sh
    #kops edit ig --name=demo.k8s.ipaperhub.com master-us-east-2a 
    ```

    *Note: change machineType: t3.medium to t2.micro for Free*

1. Create kubernetes cluser

    ```sh
    #kops update cluster demo.k8s.ipaperhub.com --yes
    ```

1. Validate your cluster
     ```sh
     #kops validate cluster --wait 10m
     ```

     To list nodes
     ```sh
     #kubectl get nodes --show-labels
     ```

     ssh to the master and change hostname to avoid confusing:

     ```sh
     #ssh -i ~/.ssh/id_rsa ubuntu@api.demo.k8s.ipaperhub.com
     #sudo hostname k8s-master
     ```

     To delete cluster

     ```sh
     #kops delete cluster demo.k8s.ipaperhub.com --yes
     ```

#### Deploying Nginx pods on Kubernetes for testing
1. Deploying Nginx Container
    ```sh
    #kubectl run sample-nginx --image=nginx --port=80
    #kubectl get pods
    #kubectl get deployments
    ```
   
1. Expose the deployment as service. This will create an ELB in front of those 2 containers and allow us to publicly access them.
   ```sh
   #kubectl expose deployment sample-nginx --port=80 --type=LoadBalancer
   #kubectl get services -o wide
   ```

