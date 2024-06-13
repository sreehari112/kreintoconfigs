To create eks cluster and install addons




## VPC Creation  --- composition completd
1. Decide the vpc cidr range and whether to use custom network configuration or not
2. VPC Should contain 
      1. Two cidr ranges --- 10.0.0.0/22 and 100.99.0.0/16
      2. Three public subnets --- where control plane will be deployed
      3. Three private subnets --- where worker nodes will run
           To run worker nodes in private subnets create  following vpc endpoints
                1. sts
                2. ec2
                3. s3 gateway Endpoints
                4. autoscaling
                5. ecr api
                6. ecr dkr
                7. elastic loadbalancing
                8. cloudwatch (optional if you enable logging)
      4. Three Non routable subnets
      5. One Internetgateway
      6. NAT gateway deploy in on az
      7. Default security group
      8. Route tables
      9. Route table assosication

## Mandatory tags --- yet to start
1. List out the required tags in array of string


## Security groups  --- composition completed
1. Dynamic security group rules



## IRSA --- composition partially completed
1. Create dynamic irsa
    i. Condition without creating service account in the k8s cluster
    ii. Condition creating IRSA with service account deployment in k8s cluster --- completed


## Cluster Creation parameters

### IAM Role creation
1. EKS CLUSTER ROLE
2. Karpenter Worker Role
3. Karpenter Instance Profile

### cluster configuratiom  --- yet to start
1. Cluster Name
2. Kubernets version
3. Cluster service role (cluster role)
### cluster access
4. Bootstrap cluster administrator access
5. Cluster authentication mode
### secrets  encryption
6. kms enabled or not
### Cluster tags
7. mandatory tags
### Networking
8. vpc id
9. cluster subnets
10. security groups
11. choose cluster ipaddress family

### cluster endpoint access
12. public
13. public and private
14. private

### configure observability

### control plane logging

### Default addon installation
1. VPC CNI
   i. version
2. Kube-proxy
   i. version
3. CoreDNS
   i. version


### create oidc for eks cluster

### Fargate profile creation
1. Create fargate profile for karpenter


     

    