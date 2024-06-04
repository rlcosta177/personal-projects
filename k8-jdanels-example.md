## Update instance and install the development tools
```bash
sudo yum update && sudo yum upgrade -y
sudo yum group install 'Development Tools' -y
```

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

