sudo yum update && sudo yum upgrade -y
sudo yum group install 'Development Tools' -y

sudo bash -c 'echo "export PATH=$PATH:/usr/local/go/bin:/home/ec2-user/go/bin" >> /etc/profile'
source /etc/profile

create .aws/credentials && .aws/config

config file
```bash
[default]
region = us-east-1
output = json
```

credentials file
```bash
[default]
aws_access_key_id=
aws_secret_access_key=
aws_session_token=
```

