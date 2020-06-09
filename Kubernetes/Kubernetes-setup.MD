# Setup Kubernetes (K8s) Cluster on AWS


1. Create Ubuntu EC2 instance
1. install AWSCLI
   ```sh
    curl https://s3.amazonaws.com/aws-cli/awscli-bundle.zip -o awscli-bundle.zip
    apt install unzip python
    unzip awscli-bundle.zip
    #sudo apt-get install unzip - if you dont have unzip in your system
    ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
    ```

1. Install kubectl on ubuntu instance
   ```sh
   curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin/kubectl
   ```

1. Install kops on ubuntu instance (Useful to set up kubernetes cluster on aws instance)
   ```sh
    curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
    chmod +x kops-linux-amd64
    sudo mv kops-linux-amd64 /usr/local/bin/kops
    ```
1. Create an IAM user/role  with Route53, EC2, IAM and S3 full access

1. Attach IAM role to ubuntu instance
   ```sh
   # Note: If you create IAM user with programmatic access then provide Access keys. Otherwise region information is enough
   aws configure
    ```

1. Create a Route53 private hosted zone (you can create Public hosted zone if you have a domain)
   ```sh
   Routeh53 --> hosted zones --> created hosted zone  
   Domain Name: vinay.in
   Type: Private hosted zone for Amzon VPC
   ```

1. create an S3 bucket
   ```sh
    aws s3 mb s3://demo.k8s.vinay.in
    In case of issue To delete: aws s3 rb s3://demo.k8s.vinay.in or aws s3 rb s3://demo.k8s.vinay.in --force
    delete attached cluster prior deleting S3 bucket 
   ```
1. Expose environment variable:
   ```sh
    export KOPS_STATE_STORE=s3://demo.k8s.vinay.in
   ```

1. Create sshkeys before creating cluster
   ```sh
    ssh-keygen
   ```

1. Create kubernetes cluster definitions on S3 bucket
   ```sh
   kops create cluster --cloud=aws --zones=ap-south-1b --name=demo.k8s.vinay.in --dns-zone=vinay.in --dns private 
    ```

1. Create kubernetes cluser (to set up cluster it takes 5 to 10 mins)
    ```sh
    kops update cluster demo.k8s.vinay.in --yes
    ```

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
     kops delete cluster demo.k8s.vinay.in --yes
    ```
1. Important
   ```sh
   Changes may require instances to restart: kops rolling-update cluster
   
   Suggestions:
   list clusters with: kops get cluster
   edit this cluster with: kops edit cluster demo.k8s.vinay.in
   edit your node instance group: kops edit ig --name=demo.k8s.vinay.in nodes
   edit your master instance group: kops edit ig --name=demo.k8s.vinay.in master-ap-southeast-1b
   
   Suggestions:
   validate cluster: kops validate cluster
   list nodes: kubectl get nodes --show-labels
   ssh to the master: ssh -i ~/.ssh/id_rsa admin@api.demo.k8s.vinay.in
   the admin user is specific to Debian. If not using Debian please use the appropriate user based on your OS.
   read about installing addons at: https://github.com/kubernetes/kops/blob/master/docs/operations/addons.md.
   
   To login to Kubernetes Master :::: ssh -i ~/.ssh/id_rsa admin@api.dev.k8s.vinay.in
   To check public IP: curl http://checkip.amazonaws.com
   To login into Kubernetes master from Ansible server: [ansadmin@ip-172-31-3-98 .ssh]$ ssh -i id_rsa root@"PublicIP"
   ```
  
#### Deploying Nginx pods on Kubernetes

1. Deploying Nginx Container
    ```sh
    kubectl run sample-nginx --image=nginx --replicas=2 --port=80
    # kubectl run simple-devops-project --image=vinaybhuria/simple-devops-image --replicas=2 --port=8080
    kubectl get pods
    kubectl get deployments
   ```
1. Expose the deployment as service. This will create an ELB in front of those 2 containers and allow us to publicly access them.
   ```sh
   kubectl expose deployment sample-nginx --port=80 --type=LoadBalancer
   # kubectl expose deployment simple-devops-project --port=8080 --type=LoadBalancer
   kubectl get services -o wide
   ```
