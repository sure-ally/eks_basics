# eks_basics
AWS EKS Basics

Create EKS cluster using CLI:

eksctl create cluster --name=eksdemo1 --region=us-east-1 --zones=us-east-1a,us-east-1b --without-nodegroup

eksctl utils associate-iam-oidc-provider --region us-east-1 --cluster eksdemo1 --approve

Node Group - Public:
eksctl create nodegroup --cluster=eksdemo1 --region=us-east-1 --name=eksdemo1-ng-public1 --node-type=t3.medium  --nodes=2 --nodes-min=2 --nodes-max=4 --node-volume-size=20 --ssh-access --ssh-public-key=kube-demo --managed --asg-access --external-dns-access --full-ecr-access --appmesh-access --alb-ingress-access

Node Group - Private:
eksctl create nodegroup --cluster=eksdemo1 --region=us-east-1 --name=eksdemo1-ng-private1 --node-type=t3.medium  --nodes=2 --nodes-min=2 --nodes-max=4 --node-volume-size=20 --ssh-access --ssh-public-key=kube-demo --managed --asg-access --external-dns-access --full-ecr-access --appmesh-access --alb-ingress-access --node-private-networking

(NOTE: create key pair: kube-demo)

Update sg of ec2 to have all traffic --> node security group (search in nodes or cloudformation stack e.g: eksctl-eksdemo1-nodegroup-eksdemo1-ng-public1-remoteAccess)
    --> All traffic - 0.0.0.0/0

Install EBS CSI driver:
    Get the node instance role: kubectl -n kube-system describe configmap aws-auth
    Create Policy: IAM policy and attach to Node IAM role:
   
        {
        "Version": "2012-10-17",
        "Statement": [
            {
            "Effect": "Allow",
            "Action": [
                "ec2:AttachVolume",
                "ec2:CreateSnapshot",
                "ec2:CreateTags",
                "ec2:CreateVolume",
                "ec2:DeleteSnapshot",
                "ec2:DeleteTags",
                "ec2:DeleteVolume",
                "ec2:DescribeInstances",
                "ec2:DescribeSnapshots",
                "ec2:DescribeTags",
                "ec2:DescribeVolumes",
                "ec2:DetachVolume"
            ],
            "Resource": "*"
            }
        ]
        }

    Attach policy to role
    Install CSI driver: 
        Verify kubectl version > 1.14: kubectl version --client --short
        Deploy EBS CSI Driver: kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=master"
        Verify ebs-csi pods running: kubectl get pods -n kube-system

# To connect from client:
# aws eks --region us-east-1 update-kubeconfig --name eksdemo1


#
curl -o iam_policy_latest.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json

aws iam create-policy --policy-name skyd-AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy_latest.json
arn:aws:iam::359524518288:policy/skyd-AWSLoadBalancerControllerIAMPolicy

eksctl create iamserviceaccount --cluster=eksdemo1 --namespace=kube-system --name=aws-load-balancer-controller --attach-policy-arn=arn:aws:iam::359524518288:policy/skyd-AWSLoadBalancerControllerIAMPolicy --override-existing-serviceaccounts --approve

get a temporary token of login using the

# kubectl create token aws-load-balancer-controller

helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=eksdemo1 --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set region=us-east-1 --set vpcId=vpc-0213e38ddcdb62417 --set image.repository=602401143452.dkr.ecr.us-east-1.amazonaws.com/amazon/aws-load-balancer-controller

kubectl -n kube-system get sa