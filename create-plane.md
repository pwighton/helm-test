# Creating 'Command Plane'

Following along [here]

### Create cloud9 workspace
  - https://us-west-2.console.aws.amazon.com/cloud9/home?region=us-east-1
  - Create environment 

### Install kubectl locally, update awscli
```
sudo curl --silent --location -o /usr/local/bin/kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.15.10/2020-02-22/bin/linux/amd64/kubectl
sudo chmod +x /usr/local/bin/kubectl
sudo pip install --upgrade awscli && hash -r
```

### Install jq, envsubst (from GNU gettext utilities) and bash-completion
```
sudo apt install jq gettext bash-completion
```
or
```
sudo yum -y install jq gettext bash-completion
```

### Make sure all binaries are ok
```
for command in kubectl jq envsubst aws
  do
    which $command &>/dev/null && echo "$command in path" || echo "$command NOT FOUND"
  done
```

### Enable bash completion
```
kubectl completion bash >>  ~/.bash_completion
. /etc/profile.d/bash_completion.sh
. ~/.bash_completion
```
    
### Create IAM role
Follow [this link](https://console.aws.amazon.com/iam/home#/roles$new?step=review&commonUseCase=EC2%2BEC2&selectedUseCase=EC2&policies=arn:aws:iam::aws:policy%2FAdministratorAccess) to create a role with admin access:
  - Select `AWS service`
  - Use case: `EC2`
  - Next: permissions
  - check AdministratorAccess
  - Next: Tags
  - {'Name': 'PW-EKS-test-20200508')
  - Role Name: `eksworkshop-admin`
  - Create Role

Attach role to workspace
  - https://eksworkshop.com/020_prerequisites/workspaceiam/

## Config AWS cli
```
export ACCOUNT_ID=$(aws sts get-caller-identity --output text --query Account)
export AWS_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')
```

Save to bash profile
```
echo "export ACCOUNT_ID=${ACCOUNT_ID}" | tee -a ~/.bash_profile
echo "export AWS_REGION=${AWS_REGION}" | tee -a ~/.bash_profile
aws configure set default.region ${AWS_REGION}
aws configure get default.region
```

## Validate IAM role
```
aws sts get-caller-identity --query Arn | grep eksworkshop-admin -q && echo "IAM role valid" || echo "IAM role NOT valid"
```


### Create an SSH key 

https://eksworkshop.com/020_prerequisites/sshkey/
"This key will be used on the worker node instances to allow ssh access if necessary."

```
ssh-keygen
```

Upload public key to ec2 region
```
aws ec2 import-key-pair --key-name "eksworkshop" --public-key-material file://~/.ssh/id_rsa.pub
```

### Create AWS KMS Custom managed Key
"Create a CMK for the EKS cluster to use when encrypting your Kubernetes secrets"
```
aws kms create-alias --alias-name alias/eksworkshop --target-key-id $(aws kms create-key --query KeyMetadata.Arn --output text)
```

Retreive ARN of CMK to input into create cluster command
```
export MASTER_ARN=$(aws kms describe-key --key-id alias/eksworkshop --query KeyMetadata.Arn --output text)
```

Save Var:
```
echo "export MASTER_ARN=${MASTER_ARN}" | tee -a ~/.bash_profile
```

### Install eksctl
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv -v /tmp/eksctl /usr/local/bin
eksctl completion bash >> ~/.bash_completion
. /etc/profile.d/bash_completion.sh
. ~/.bash_completion
```

### Install Helm
```
curl -sSL https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

```
helm completion bash >> ~/.bash_completion
. /etc/profile.d/bash_completion.sh
. ~/.bash_completion
source <(helm completion bash)
```
