# Helm-test

Trying to learn helm.

## Setup

1) [Create Kubernets 'command plane' Amazon EKS](create-plane.md)

## Deploy test cluster

```
cat ./eks-test-cluster.yml |sed 's/MASTER_ARN/'"${MASTER_ARN}"'/' > eks-test-cluster.deploy.yml
```



