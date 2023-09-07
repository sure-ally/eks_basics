# eks_basics
AWS EKS Basics

Create EKS cluster using CLI:

eksctl create cluster --name=eksdemo1 --region=us-east-1 --zones=us-east-1a,us-east-1b --without-nodegroup

eksctl utils associate-iam-oidc-provider --region us-east-1 --cluster eksdemo1 --approve

eksctl create nodegroup --cluster=eksdemo1 --region=us-east-1 --name=eksdemo1-ng-public1 --node-type=t3.medium  --nodes=2 --nodes-min=2 --nodes-max=4 --node-volume-size=20 --ssh-access --ssh-public-key=kube-demo --managed --asg-access --external-dns-access --full-ecr-access --appmesh-access --alb-ingress-access

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
        Verify kubectl versio > 1.14: kubectl version --client --short
        Deploy EBS CSI Driver: kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=master"
        Verify ebs-csi pods running: kubectl get pods -n kube-system



