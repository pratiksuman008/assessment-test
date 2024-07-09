

## Prerequisites:

    AWS account
    Kubectl installed
    AWS CLI installed
    Terraform installed

  <br />

## Objectives:

    Set up AWS credentials
    Install kubectl (if not already done)
    Configure working directory
    Edit .tf files to create Kubernetes cluster infrastructure
    Deploy the Kubernetes cluster
    Deploy fortune-api on the cluster

  <br />
    <br />


## Files and Folders:
```
terraform-files:
    - main.tf  - contains the main set of configuration for the module.
    - variables.tf  - contains the variable definitions for the module.
    - outputs.tf  - contains the output definitions for the module.
    - terraform.tf - contains the providers for the module.
deployment.yaml - provides declarative template for Pods.
```
 <br />
  <br />


### Steps to Docker build image and push to dockerhub

docker build --platform=linux/arm64  -t pratiksuman008/fortune-api .

docker push pratiksuman008/fortune-api

  <br />
  <br />

### Set up the AWS account
-- link the account to the AWS CLI.

For this part, i needed :

```
   1. AWS Access Key ID.
   2. AWS Secret Access Key.
   3. Default region name.
   4. Default output format.
```
  <br />

***aws configure***

AWS Access Key ID [None]:

AWS Secret Access Key [None]:

Default region name [None]:

Default output format [None]:

<br />
<br />

### Creating an Amazon Elastic Kubernetes Service Cluster using Terraform and the following AWS resources:

```
1. AWS Virtual Private Cloud (VPC).
2. Two public and Two private AWS Subnets.
3. One AWS Route Table.
4. Two AWS Route Table Association.
5. AWS Internet Gateway attached to the VPC.
6. AWS EKS Cluster. It will have one master node to manage the Kubernetes application.
7. AWS EKS Node Groups.
8. AWS Security Groups.
```


<br />


```
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "20.8.5"

  cluster_name    = local.cluster_name
  cluster_version = "1.30"

  cluster_endpoint_public_access           = true
  enable_cluster_creator_admin_permissions = true



  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  eks_managed_node_group_defaults = {
    ami_type = "AL2_x86_64"

  }

  eks_managed_node_groups = {
    one = {
      name = "node-group-1"

      instance_types = ["t2.micro"]
      capacity_type  = "SPOT"

      min_size     = 1
      max_size     = 1
      desired_size = 1
    }
    two = {
      name = "node-group-2"

      instance_types = ["t2.micro"]

      min_size     = 1
      max_size     = 1
      desired_size = 1
    }
  }
}
```

***terraform plan***

***terraform apply***

***Outputs:***
  - cluster_endpoint          = "https://64206471E08D9371CEF7C514F4C76A82.gr7.us-east-2.eks.amazonaws.com"
  - cluster_name              = "test-eks-FXSX8qWj"
  - cluster_security_group_id = "sg-06074dba698c5990f"
  - region                    = "us-east-2"

  <br />

### Configure kubectl with EKS API Server credential

Create or update a kubeconfig file for the cluster. Replace region-code with the AWS Region that the cluster is in and replace [EKS_Cluster_Name] with the name of the cluster.\
By default, the resulting configuration file is created at the default kubeconfig path (.kube) in the home directory or merged with an existing config file at that location. It can be specified with another path with the --kubeconfig option.

aws eks --region [EKS_Region] update-kubeconfig --name [EKS_Cluster_Name]


   <br />
   <br />

### Kubernetes Deployment
Deploying the fortune-api image as deployment object in Kubernetes with one pod replica.

_Note: deployed in Default namespace_

***kubectl apply -f deployment.yaml***


```***---
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: fortune-api-deployment
    labels:
      app: fortune-api
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: fortune-api
    template:
      metadata:
        labels:
          app: fortune-api
      spec:
        containers:
        - name: fortune-api
          image: pratiksuman008/fortune-api
          ports:
          - containerPort: 8080***
```
```
kubectl get pods
NAME                                     READY   STATUS    RESTARTS   AGE
fortune-api-deployment-7dbff945b-q4z6w   1/1     Running   0          39s
