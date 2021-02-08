# pi-sip
Detailed explaination of setting up a sip server on CentOS 7 and sip client on Raspberry Pi

**Note: submit [new issues](https://github.com/nikhiljohn10/pi-sip/issues/new) for corrections**

## CentOS 7/8 setup

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
ssh-keygen -t rsa -b 4096 -C "YOUR_EMAIL_ID" # Generating SSH Key
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
(local)$ ssh-copy-id pisip@SERVER_IP # Adding local SSH Key to server
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
```

### Installing Maria DB

```
yum install epel-release
yum update
yum install mariadb-server mariadb
systemctl start mariadb
mysql_secure_installation
systemctl enable mariadb
```

## Kamailio server setup

```
yum install kamailio kamailio-ldap kamailio-mysql kamailio-postgres kamailio-debuginfo kamailio-xmpp kamailio-unixodbc kamailio-utils kamailio-tls kamailio-outbound kamailio-gzcompr kamailio-presence gdb
vim /etc/kamailio/kamctlrc # DBENGINE=MYSQL
kamdbctl create
kamctl add test1 password1 # If you want to dial number instead on client, simply use that number as username here.
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

- username: test1@SERVER_IP:5060
- password: password1
- outbound_proxy: SERVER_IP:5060
- TLS protocols: TLSv1

You can use any SIP clients like,

- [SIP SIMPLE Client](http://sipsimpleclient.org/projects/sipsimpleclient/wiki/SipInstallation)
- [Blink](http://icanblink.com/) - Try `sudo apt update && sudo apt upgrade && sudo apt install blink` on ubuntu based linux
- [Jitsi](https://jitsi.org/Main/Download) - Installation is little tricky when you can't find dependancies. But after some searching you can install it. This [link](https://github.com/jitsi/jitsi-meet/wiki/Debian-installation) may help you.
- [MicroSIP](http://www.microsip.org/)
- [SessionChat](https://itunes.apple.com/us/app/sessionchat-sip-softphone/id362501443?mt=8) - This works well on iOS voice call to server

## Building a custom SIP client

**Targets :**
- Use Raspberry Pi as client system to dial and reveice calls
- Use [SIP SIMPLE Client SDK](https://github.com/AGProjects/python-sipsimple) for connecting to server
- Use a specific Bluetooth device as permanent communication device
- Modify a case to make dialer panel

---


**References**

[Initial Server Setup with CentOS 7](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-centos-7)

[Additional Recommended Steps for New CentOS 7 Server](https://www.digitalocean.com/community/tutorials/additional-recommended-steps-for-new-centos-7-servers)

[Howto: Kamailio SIP proxy with hosted NAT traversal on Debian Wheezy](https://richardskingdom.net/howto-kamailio-sip-proxy-nat-debian-wheezy)
