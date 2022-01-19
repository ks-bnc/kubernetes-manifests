### Create on-demand nginx deployment on AWS with ArgoCD

With the following description and steps you can create your own Kubernetes cluster with EKS, and with the additional resources (nginx-deployment.yml & nginx-service.yml) you can deploy your own NGINX service, with Classic AWS Loadbalancer.

1. Create IAM role 
   1. Services - Security - IAM - Users
   2. Add user - Name k8-admin - Credential type is Access Key 
   3. Attach existing policy directly - Administrator Access
   4. Create user
--- 
2. Create Cluster (EC2 instance with AWS CLI v2, eksctl, kubectl)
   1. Create instance (aws linux ami, t2.medium, enable auto public ip assign, create keypair)
   2. ssh to instance (set private-key to 400, ssh -i <keyfile> ec2-user@<ip>)
   3. Initialize the instance
        1. `curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"`
        2. `unzip awscliv2.zip`
        3. `sudo ./aws/install`
        4. `aws --version`
        5. `which aws`
        6. `sudo ./aws/install --bin-dir /usr/bin --install-dir /usr/bin --update`
        7. `aws configure` #Set your access keys what you created in the IAM role section
        8. `curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/linux/amd64/kubectl`
        9. `chmod +x ./kubectl`
        10. `mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin`
        11. `kubectl version --short --client`
        12. `curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp`
        13. `sudo mv /tmp/eksctl /usr/local/bin`
        14. `eksctl version`
--- 
3. Create EKS cluster
    1. `eksctl create cluster --name nginx-cluster --version 1.21 --region us-east-1 --nodegroup-name standard-workers --node-type t3.medium --nodes 3 --nodes-min 1 --nodes-max 4 --managed`
    2. `eksctl get cluster`
--- 
4. Create NGINX deployment

    1.  `vim nginx-deployment.yml`
    2.  `vim nginx-service.yml`
    3.  `kubectl apply -f ./nginx-deployment.yml`
    4.  `kubectl get deployment`
    5.  `kubectl apply -f ./nginx-service.yml`
    6.  `kubectl get service`
    7.  `kubectl get pods`
    8.  `curl <lb_endpoint>`

5. ArgoCD 
   1.  `kubectl create namespace argocd`
   2.  `kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`
   3.  `kubectl get pods -n argocd`
   4.  `kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'`
   5.  `kubectl get svc -n argocd`
   6.  `kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo`