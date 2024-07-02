ec2:t2.large, 25gb
Install Jenkins, Docker, Sonarqube , trivy 
sonar-token:squ_657070d51879bd83f0494393a580bf9c8011351c

install kubectl,eksctl,awscli


# configure cluster
get iam role
name: name
acces-key:AKIA3FLDZWXYJ1dJFD5
secret-key:mPUHZlhSt4/ErbMWu8GPjksdk1LjtcGoPkLUJAPdL9

# create eks cluster
eksctl create cluster --name=EKS-1 \
                      --region=us-east-1 \
                      --zones=us-east-1a,us-east-1b \
                      --without-nodegroup

# Associate IAM OIDC Provider to accept the IAM credential in EKS cluster:
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster EKS-2 \
    --approve
    
# Create EKS Node Group as worker
eksctl create nodegroup --cluster=EKS-2 \
                       --region=us-east-1 \
                       --name=node2 \
                       --node-type=t2.medium \
                       --nodes=3 \
                       --nodes-min=2 \
                       --nodes-max=4 \
                       --node-volume-size=20 \
                       --ssh-access \
                       --ssh-public-key=DevOps \
                       --managed \
                       --asg-access \
                       --external-dns-access \
                       --full-ecr-access \
                       --appmesh-access \
                       --alb-ingress-access
    
# Create a namespace for app deployment isolation
kubectl create namespace webapps

# service-account-token

    
    
    
    
    
    
    
    
