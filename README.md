# k8-ingress
`1. Install OIDC provider
```
https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/deploy/installation/

eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster roboshop-dev \
    --approve

aws eks update-kubeconfig --region us-east-1 --name roboshop-dev
```
2. Download & Create IAM policy
```
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.16.0/docs/install/iam_policy.json

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
```
3. Create an IAM role and Kubernetes ServiceAccount for the LBC. Use the ARN from the previous step.
```
eksctl create iamserviceaccount \
--cluster=roboshop-dev \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::406682759639:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region us-east-1 \
--approve
```
4. Install drivers

Install helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
chmod 700 get_helm.sh
./get_helm.sh
```
Add eks-charts repo
```
helm repo add eks https://aws.github.io/eks-charts
```
Install loadbalancer contoller
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=roboshop-dev --set serviceAccount.create=false
--set serviceAccount.name=aws-load-balancer-controller
```

Check drivers are running
```
k get po -n kube-system|grep aws
aws-load-balancer-controller-5449867884-nhsx8   1/1     Running   0          2m18s
aws-load-balancer-controller-5449867884-p9cwq   1/1     Running   0          2m18s
```

