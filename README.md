# KOPS
How to use Kops to ensure K8s cluster HA

### Creating a K8s cluster with kops:

1. Create an user with its sets of keys (AWS user created via IAM) with S3/R53/EC2/VPC/IAM admin access role permissions.

2. Delegate authority for the subdomain you choose (get your subdomain nameservers and add a NS record to the parent hosted zone that contains those 4 NS records so it can know where to resolve your subdomain).

3. Create ssh key at your computer (you'll need it to ssh your nodes):
```
ssh-keygen -f ~/.ssh/<key-name> -t rsa
```
4. Create a S3 bucket (at your AWS consola or via CLI) and set the environment variable so Kops can manage files and know where to store cluster's configs:
You can export the environment variable:
```
export KOPS_STATE_STORE=s3://your-bucket
```
Or you can set a bash alias (you'll need it later on):
```
alias <your-alias> = 'kubectl config use-context <your-cluster-hosted-zone>; export KOPS_STATE_STORE=s3://<your-bucket-name>; export AWS_PROFILE=<your-aws-profile>'
```
5. Install Kops (you must have kubectl installed):
```
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64 && chmod +x kops-linux-amd64 && mv kops-linux-amd64 /usr/local/bin/kops
```

6. Set the key that Kops will use for your cluster (the one created at step 3):
```
kops create secret --name <your-subdomain.example.com> sshpublickey admin -i ~/.ssh/<yourkey>.pub
```

7. Create cluster configuration (it's like a dry-run):
```
kops create cluster --node-count=2 --node-size=t2.medium --node-volume-size=8 --master-volume-size=8 --zones=us-east-1a --name=yoursubdomain.example.com --master-size=t2.medium --master-count=1 --dns-zone=yoursubdomain.example.com --ssh-public-key=~/.ssh/yourkey.pub
```
 
 #### Note:

I had an issue last week (mid sept. 2018) where the cluster failed, I don't know why (didn't have the time to get into that) I use t2.micro/nano because I'm just doing some tests, and after changing node/master size to t2.small, worked.
 
8. Create the cluster (it only adds --yes to the previous command (that's why I called it dry run)):
```
kops update cluster yoursubdomain.example.com --yes
```

Optional: Check every 1s the status of the cluster creation:
```
watch -n1 'kops validate cluster'
```

#### Edit the cluster with Kops  (ig stands for instance group):
This will adjust the ig configuration but not the launch configuration at AWS:
```
kops edit ig 
```
Apply these changes:
```
kops update cluster --yes
```
After that kops rolling-update cluster should show you that updates could be performed.

Execute rolling-update with the --yes parameter and your master instance size should be changed:
```
kops rolling-update --yes
```
