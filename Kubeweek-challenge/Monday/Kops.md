### Install openjdk

```
sudo apt update && sudo apt upgrade -y
java -version
sudo apt-get install openjdk-17-jdk
java -version
```

### Set environment variable
```
vi ~/.bashrc
export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java))))
export PATH=$PATH:$JAVA_HOME/bin
source ~/.bashrc

echo $JAVA_HOME
echo $PATH
```

### Install kubectl
#### Download the latest release
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

#### Download the kubectl checksum file
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

##### Validate kubectl binary against the checksum file
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check

#### install kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

#### If you do not have root access on the target system, you can still install kubectl to the ~/.local/bin directory
chmod +x kubectl
mkdir -p ~/.local/bin
mv ./kubectl ~/.local/bin/kubectl

#### Check the version
kubectl version --client
```

### Install KOPS
```
curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops
sudo mv kops /usr/local/bin/kops
```

### Install awscliv2
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
### Create IAM Role

#### create iam role3 with access key and secret access key

#### Attach Policies
```
AmazonEC2FullAccess
AmazonRoute53FullAccess
AmazonS3FullAccess
IAMFullAccess
AmazonVPCFullAccess
AmazonSQSFullAccess
AmazonEventBridgeFullAccess
```

### configure access key and secret access key
```
aws configure

cd .aws
cat config
cat creedentials
```

### verify if everything working fine or not
```
aws sts get-caller-identity
```

### Create bucket
```
aws s3api create-bucket \
    --bucket prefix-example-com-state-store \
    --region us-east-1
OR
bucket_name=example-26-09-2024

aws s3api create-bucket \
    --bucket ${bucket_name} \
    --region us-east-1
```
### Note: We STRONGLY recommend versioning your S3 bucket in case you ever need to revert or recover a previous state store.
```
aws s3api put-bucket-versioning --bucket prefix-example-com-state-store  --versioning-configuration Status=Enabled
```

### Create cluster name with variable:
#### We did not have purchase any domain, So used local domain as in variable.
```
export KOPS_CLUSTER_NAME=myfirstcluster.example.com
export KOPS_STATE_STORE=s3://prefix-example-com-state-store
or
export KOPS_STATE_STORE=s3://${bucket_name}
```

### Check availability zones
```
aws ec2 describe-availability-zones --region us-west-2
```

### Create ssh-key
```
ssh-keygen -t rsa
```

### Create cluster

```
kops create cluster \
    --name=${KOPS_CLUSTER_NAME} \
    --cloud=aws \
    --node-count=2
    --zones=us-west-2a \
    --ssh-public-key=/home/user/.ssh/id_rsa.pub
    --discovery-store=s3://prefix-example-com-oidc-store/${bucket_name}/discovery
```

### Some of commands used to edit or update cluster.
```
kops edit cluster --name ${KOPS_CLUSTER_NAME}
kops edit ig --name ${KOPS_CLUSTER_NAME} node-us-east-1a
kops edit ig --name ${KOPS_CLUSTER_NAME} master-us-east-1a
kops update cluster --name ${KOPS_CLUSTER_NAME} --yes
```

### Comnmands used in KOPS to get info.
```
kops get cluster
kops validate cluster
kubectl get nodes --show-labels
kubectl get pods
kubectl get nodes -o wide
#### Connect with node
ssh -i ~/.ssh/id_rsa ubuntu@external-ip-master-node  
```

### After validation if you face any issue run below command

```
kops export kubecfg --admin
kops validate cluster        #same as "kubectl get all"

#### after coming back from pod it will remove automatically.

kubectl run pod1 --rm --restart=Never --image=imagename -i -t -- sh

#### Describe pod
kubectl describe pod

#### Delete cluster
kops delete cluster --name ${KOPS_CLUSTER_NAME}  --yes

```
### Create high_availability cluster
```
#### Create a cluster using public network topology:

kops create cluster \
    --node-count 3 \
    --zones us-west-2a,us-west-2b,us-west-2c \
    --master-zones us-west-2a,us-west-2b,us-west-2c \
    hacluster.example.com

#### Create a cluster using private network topology:

kops create cluster \
    --node-count 3 \
    --zones us-west-2a,us-west-2b,us-west-2c \
    --master-zones us-west-2a,us-west-2b,us-west-2c \
    --topology private \
    --networking <provider> \
    ${KOPS_CLUSTER_NAME}

#### Multiple masters in the same AZ    

kops create cluster \
    --node-count 3 \
    --master-count 3 \
    --zones cn-north-1a,cn-north-1b \
    --master-zones cn-north-1a,cn-north-1b \
    hacluster.k8s.local
```
### Document References
- https://kops.sigs.k8s.io/getting_started/install/
- https://kops.sigs.k8s.io/getting_started/aws/
- https://kops.sigs.k8s.io/operations/high_availability/#advanced-example
