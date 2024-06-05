## Update instance and install the development tools
```bash
sudo yum update -y
sudo yum group install 'Development Tools' -y
```

2.git clone do repo do prof

- git clone https://github.com/jdmedeiros/k8s.git
- cd k8s
- git checkout original_v126_update
- git pull

3. aws cli install

- https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

## 0-install-kubernetes-tools FIX 
```bash
sudo bash -c 'echo "export PATH=$PATH:/usr/local/go/bin:/home/ec2-user/go/bin" >> /etc/profile'
source /etc/profile
go install github.com/cloudflare/cfssl/cmd/cfssl@latest
go install github.com/cloudflare/cfssl/cmd/cfssljson@latest
```

## Create .aws/credentials && .aws/config

- config file
```bash
[default]
region = us-east-1
output = json
```

- credentials file
```bash
[default]
aws_access_key_id=
aws_secret_access_key=
aws_session_token=
```

sudo nano /etc/profile -> add :/usr/local/bin
source 10, 7, 5, 4, 1
kubectl get all
kubectl get deployments.apps 
