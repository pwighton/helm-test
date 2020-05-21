# Helm-test

Trying to learn helm.

Refs
  - https://eksworkshop.com/030_eksctl/
  - https://docs.gitlab.com/charts/

## Setup

1) [Create Kubernets 'command plane' Amazon EKS](create-plane.md)

### Deploy test cluster

```
cat ./eks-test-cluster.yml | sed 's|MASTER_ARN|'"${MASTER_ARN}"'|' > eks-test-cluster.deploy.yml
eksctl create cluster -f eks-test-cluster.deploy.yml
```

To delete the cluster:
```
eksctl delete cluster --name=eksworkshop-eksctl
```

If encounter issues, check CloudFormation Console or
```
eksctl utils describe-stacks --region=us-east-1 --cluster=eksworkshop-eksctl
```

### Test
see [here](https://eksworkshop.com/030_eksctl/test/)

```
kubectl get nodes
```

### Save Worker Role Name

```
STACK_NAME=$(eksctl get nodegroup --cluster eksworkshop-eksctl -o json | jq -r '.[].StackName')
ROLE_NAME=$(aws cloudformation describe-stack-resources --stack-name $STACK_NAME | jq -r '.StackResources[] | select(.ResourceType=="AWS::IAM::Role") | .PhysicalResourceId')
echo "export ROLE_NAME=${ROLE_NAME}" | tee -a ~/.bash_profile
```

## Deploy gitlab

Following along [here](https://docs.gitlab.com/charts/quickstart/index.html) and [here](https://stackoverflow.com/questions/51511547/empty-address-kubernetes-ingress) to enable the ingress for IP addresses 

```
helm install gitlab gitlab/gitlab \
  --set global.hosts.domain=dev.corticometrics.com \
  --set certmanager-issuer.email=paul@corticometrics.com
```
