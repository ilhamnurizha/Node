# How To Protect Your Node Server From DDoS or Brute Force

## Change ssh password
```
Use Strong password with minium 8 character and combine with numeric and symbol
```
Ex: Qwerty1234!@#

## Firewall configuration (ufw)
Install UFW if you use ubuntu Server
```
sudo apt ufw install -y
```
Closed all port, just open port what you needed
```
# Allow ssh connection
sudo ufw allow ssh
# SSH port
sudo ufw allow 22
# # API server port
sudo ufw allow 1317
# Ports for p2p connection, RPC server, ABCI
sudo ufw allow 26656:26658/udp
# Prometheus port
sudo ufw allow 26660
# Port for pprof listen address
sudo ufw allow 6060
# Address defines the gRPC server address to bind to.
sudo ufw allow 9090
# Address defines the gRPC-web server address to bind to.
sudo ufw allow 9091
```
Enable and check ufw rule
```
ufw enable
ufw status
```
## Change default port ssh
change port for default ssh configuration, use port number whatever you want, i use port 2222 for example
```
nano /etc/ssh/sshd_config
```
Locate line that read as follows
```
#Port 22
```
Change it into
```
Port 2222
```
Allow ufw rule
```
sudo ufw allow 1234/tcp
sudo ufw deny 22
```
Restart sshd service
```
sudo systemctl restart sshd
```

## Using SSH key login
It is also highly recommended to set up an SSH key login instead of a password. This is a more reliable method of protection than a password.

Now we will enable ssh key login, and disable password login. 

This guide is for those who have **Linux/macOS** installed on their local computer. If you have a different system, such as **windows**, use [[this instruction](https://surftest.gitbook.io/axelar-wiki/english/security-setup/ssh-key-login-+-disable-password)].

Generate ssh keys
Unix has a built-in key generator. Launch the terminal on PC and enter:
```
ssh-keygen
```
> It will be possible **to set a password for SSH keys**. This will additionally protect you from possible key theft. The main thing is to write down the password in a safe place. If you do not want to set a password, press **Enter**.
Done, the keys are created and stored in the folder `~/.ssh/` (for example `/home/user/.ssh/`)
- `~/.ssh/id_rsa` - private key. We should leave it on PC.
- `~/.ssh/id_rsa.pub` - public key. Must be on the server.

Uploading the public key to the server
On Unix systems for this you could open a terminal on PC and enter the command:
```
ssh-copy-id root@ххх.ххх.ххх.ххх
```
where:
- `root` - the user we want to access the server using ssh keys.
- `ххх.ххх.ххх.ххх` - server ip address.

```
ssh-copy-id -i /home/user/.ssh/id_rsa_test.pub -p 2222 root@ххх.ххх.ххх.ххх
```
Flags:
- `-i /home/user/.ssh/id_rsa_test.pub` - specify the path to your public key.
- `-p 2222`  - specify your ssh port.

## Disabling password login.
Make sure that the key is securely stored and you will not lose it, because you will no longer log in with the password (only through the console in the provider's personal account, or through VNS).

Log in to the server, then open the configuration file for editing `/etc/ssh/sshd_config`:
```
sudo nano /etc/ssh/sshd_config
```
Find the line there `PasswordAuthenticatin yes`. You need to set its value to `No`:
```
PasswordAuthentication no
```
Restart ssh service:
```
sudo systemctl restart sshd
```
