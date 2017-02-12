# pi-sip
Detailed explaination of setting up a sip server on CentOS 7 and sip client on Raspberry Pi

## Centos setup

```
(local)$ ssh root@SERVER_IP
passwd root
adduser pisip # Your can replace 'pisip' with any username you prefer
passwd pisip
gpasswd -a pisip wheel # Gives sudo access to user
sudo visudo # %wheel  ALL=(ALL)       NOPASSWD: ALL ; Need to change this back after configuring server.
vi /etc/sysconfig/selinux # SELINUX=disabled ; This depends on if you want extra layer of security or not.
reboot
```
After rebooting,

```
(local)$ ssh pisip@SERVER_IP
sestatus # se test
sudo whoami # sudo test
ls -al ~/.ssh # ssh key test
ssh-keygen -t rsa -b 4096 -C "YOUR_EMAIL_ID"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
(local)$ ssh-copy-id pisip@SERVER_IP
sudo su
yum update
yum install vim firewalld ntp
vim /etc/ssh/sshd_config
systemctl reload sshd
systemctl start firewalld
systemctl enable firewalld
firewall-cmd --permanent --add-service=http && firewall-cmd --permanent --add-service=https
firewall-cmd --permanent --add-port=5060/tcp && firewall-cmd --permanent --add-port=5060/udp
firewall-cmd --reload
timedatectl list-timezones # Find your timezone and set it in next command
timedatectl set-timezone Asia/Kolkata # Set your timezome here from the list
systemctl start ntpd
systemctl enable ntpd
exit
```

### Installing Maria DB

```
sudo yum install epel-release
sudo yum install mariadb-server mariadb
sudo systemctl start mariadb
sudo mysql_secure_installation
sudo systemctl enable mariadb
```

## Kamailio server with yum

```
sudo su
yum update
yum install kamailio kamailio-ldap kamailio-mysql kamailio-postgres kamailio-debuginfo kamailio-xmpp kamailio-unixodbc kamailio-utils kamailio-tls kamailio-outbound kamailio-gzcompr kamailio-presence gdb
vim /etc/kamailio/kamctlrc # DBENGINE=MYSQL
kamdbctl create
kamctl add test1 password1
kamctl add test2 password2
vim /etc/kamailio/kamailio.cfg
<<
/* starting */
#!KAMAILIO
/* add following lines */
#!define WITH_MYSQL
#!define WITH_TLS
#!define WITH_AUTH
#!define WITH_USRLOCDB
#!define WITH_NAT
#!define WITH_PRESENCE
#!define WITH_ACCDB

...

>>
for f in /etc/ssl/certs/*.pem ; do cat "$f" >> /etc/kamailio/ca_list.pem ; done # Assuming host private key and certificates are stroed at /etc/ssl/certs/ as .pem files
chown kamailio:kamailio /etc/kamailio/ca_list.pem
chmod 0644 /etc/kamailio/ca_list.pem
vim /etc/kamailio/tls.cfg # Configuring TLSv1
<<
[server:default]
method = TLSv1
verify_certificate = no
require_certificate = no
private_key = /etc/ssl/certs/host_key.pem
certificate = /etc/ssl/certs/host_ca.pem
ca_list = /etc/kamailio/ca_list.pem
>>
service kamailio start # If everything is working fine, this should show a 'successfully restarted' message

```

**Following informations are required to connect to this server**
- username: test1@SERVER_IP:5060,
- password: password1,
- outbound_proxy: SERVER_IP:5060,
- TLS protocols: TLSv1
