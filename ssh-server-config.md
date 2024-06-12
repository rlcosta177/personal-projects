## Create the ssh server


```bash
sudo apt install openssh-server

sudo systemctl start ssh
sudo systemctl enable ssh

sudo ufw allow ssh
sudo ufw enable
```

- enable port forwarding in your router(22 -> 22 | or xxxx -> 22 is safer)

```bash
ssh <username>@<IP-ADDRESS>

ssh -p <custom-port> <username>@<public-ip-or-ddns-domain>
```

## Add key authentication

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

nano ~/.ssh/authorized_keys
 - paste your public key here

chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

sudo nano /etc/ssh/sshd_config
 - PubkeyAuthentication yes
 - PasswordAuthentication no
 - ChallengeResponseAuthentication no

sudo systemctl restart ssh
```

`ssh -i ~/.ssh/id_rsa <username>@<IP-ADDRESS>`
