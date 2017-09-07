# Installation of a gridFTP server

In the previous examples we showed how to ingest data into iRODS via the icommands. To transfer large data EUDAT offers the possibility to employ gridFTP to directly enter data into an iRODS zone.
Here we show how to setup a gridFTP endpoint, in module 09 we explain how to connect this gridFTP endpoint with iRODS.

## Environment
- Ubuntu 14.04 srever
- iRODS 4.1.X server

## Prerequisites
### 1. Update and upgrade if necessary
```sh
apt-get update
apt-get upgrade
```
### 2. Set firewall
```sh
sudo apt-get install iptables-persistent
```
- open ports 2811 and 50000-51000 in /etc/iptables/rules.v4
```sh
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [4538:480396]
-A INPUT -m state --state INVALID -j DROP
-A INPUT -p tcp -m tcp ! --tcp-flags FIN,SYN,RST,ACK SYN -m state --state NEW -j DROP
-A INPUT -f -j DROP
-A INPUT -p tcp -m tcp --tcp-flags FIN,SYN,RST,PSH,ACK,URG NONE -j DROP
-A INPUT -p tcp -m tcp --tcp-flags FIN,SYN,RST,PSH,ACK,URG FIN,SYN,RST,PSH,ACK,URG -j DROP
-A INPUT -p icmp -m limit --limit 5/sec -j ACCEPT
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 1248 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 1247 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 20000:20199 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 4443 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 443 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 5432 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 2811 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 50000:51000 -j ACCEPT
-A INPUT -j LOG
-A INPUT -j DROP
COMMIT
```

```sh
/etc/init.d/iptables-persistent restart
```
 For Ubuntu 16.04 you can follow [this guide](http://dev-notes.eu/2016/08/persistent-iptables-rules-in-ubuntu-16-04-xenial-xerus/) to change your default iptables configuration.
 
 To ensure the mapping from IP to hostname you might have to edit the */etc/hosts*:
```sh
hostname
echo "your.ip.num.ber <yourhostname>" >> /etc/hosts
```

## Installing the globus toolkit
- Download the package
```sh
wget http://downloads.globus.org/toolkit/gt6/stable/installers/repo/deb/globus-toolkit-repo_latest_all.deb
sudo dpkg -i globus-toolkit-repo_latest_all.deb
sudo apt-get update
```
- Install globus-data-management-client, globus-gridftp, globus-gsi
```sh
sudo apt-get install globus-data-management-client globus-gridftp globus-gsi
```

## Creating a certificate authority (CA) on the server
Installing the *globus-gsi* module will automatically create a *simple ca* on your server. 

```sh
/var/lib/globus/simple_ca

The unique subject name for this CA is:

cn=Globus Simple CA, ou=simpleCA-<hostname>, ou=GlobusTest, o=Grid
```

The certificates are automatically stored and linked in */etc/grid-security/*

- In case you want to setup your own CA follow the following steps.
```sh
sudo su -
cd /root/
grid-ca-create
```
 Follow the prompt.

- Create symlinks in */etc/grid-security*:
```sh
cd /etc/grid-security
```
```sh
ln -s /var/lib/globus/simple_ca/grid-security.conf grid-security.conf
ln -s /var/lib/globus/simple_ca/globus-host-ssl.conf  globus-host-ssl.conf
ln -s /var/lib/globus/simple_ca/globus-user-ssl.conf  globus-user-ssl.conf
```

- Request a host certificate and sign it:
```sh
grid-cert-request -host <fully.qualified.hostname> -force
```
 Make sure *\<fully.qualified.hostname\>* matches the way how to call the server from outside to transfer data.
 If you use a different hostname, users will have to add the mapping from IP to the hostname in their */etc/hosts* on their client machine.

 Sign the certificate, check it and restart the gridFTP server
```sh
grid-ca-sign -in /etc/grid-security/hostcert_request.pem -out /etc/grid-security/hostcert.pem
openssl x509 -in /etc/grid-security/hostcert.pem -text -noout
/etc/init.d/globus-gridftp-server restart
```

## Creating user certificates and editing the gridmap file
Users need to have an own user certificate in order to transfer data to the gridFTP endpoint. This is how you create and sign a user certificate.
- As user on the server that runs gridFTP create a user certificate:
```sh
grid-cert-request
```
This will create the *.globus* folder in your *home* directory and create a *request* file, the user key and an empty user certificate.

A user with sudo rights needs to sign the request and create the user certificate:
``` 
sudo grid-ca-sign -in /<path to>/.globus/usercert_request.pem -out /<path to>/.globus/usercert.pem
```
- In case the user has no own account on the gridFTP server, a sudo user can create user key and user certificate.
- The the *usercert.pem* and the *userkey.pem* needs to placed in a folder */home/user/.globus/* on the machine from which the user wants to connect to the gridFTP server (in the of this repository you need to transfer these two files to the user interface machine).

- Add the subject of the user to the gridmap file
```sh
grid-cert-info -subject
sudo grid-mapfile-add-entry -dn "/O=Grid/OU=GlobusTest/OU=simpleCA-irods4.alice/OU=Globus Simple CA/CN=alice" -ln alice
```
 The flag *-ln* specifies a user on your unix system on the server runnning gridFTP. Since we will use gridFTP to communicate to iRODS the ln-flag needs to specify the iRODS username rather than a linux user, we will adjust that later. For testing purposes, however, we will first map the subject to a linux user and later change it to the iRODS user name.

## Configuring the gridFTP endpoint
- Open the */etc/gridftp.conf* and add the control ports and some path for logging:

```sh
# globus-gridftp-server configuration file

# this is a comment

# option names beginning with '$' will be set as environment variables, e.g.
$GLOBUS_ERROR_VERBOSE 1
$GLOBUS_TCP_PORT_RANGE 50000,51000

# port
port 2811
log_level ALL
log_single "/var/log/globus-gridftp-server.log"
log_transfer "/var/log/globus-gridftp-server-transfer.log"
```

- Restart the gridFTP server
```sh
/etc/init.d/globus-gridftp-server restart
```


## Testing the gridFTP endpoint on the server
- If you have not done so install the grid certificates to the *.globus* directory of a unix user on the gridFTP server. 
```sh
mkdir .globus
cd .globus
```
- Make sure the certificates belong to the user (here *alice*)
```
sudo chown alice:alice *
```
- Become the user and initialise a proxy
```sh
grid-proxy-init
```

Try to list the */tmp* directory via gridFTP and create and copy a file to that directory.
The protocol to use is *gsiftp* and the server name needs to match the field *CN* in the host certificate.
```sh
globus-url-copy -dbg -list gsiftp://alice-server/tmp/
echo "kjsbdj" > /home/alice/test.txt
globus-url-copy file:/home/alice/test.txt gsiftp://alice-server/tmp/test.txt
```

## Hostname, fully qualified name and grid certficates

In this example we used the `hostname` to address the gridFTP server. The way to address the gridFTP server is set in the server certificates. The default setup uses the `hostname`. No matter from where you want to address and access the gridFTP endpoint on this server, you will have to use the `hostname`. When you try to get access from a different server via the internet that poses a problem. 
Machines from which you want to get access need in their */etc/hosts* file an entry that maps the fully qualified domain name or IP address of the gridFTP server to the `hostname` of the gridFTP server.
For production it is advised to create new server certificates and replace the *CN* (Common name) with the fully qualified domain name of the server (Step **Creating a certificate authority (CA) on the server**).

## Trouble shooting
When accessing the gridFTP server from outside, you might run into the problem that the server just listens on ipv6 interfaces. To change this, add the following line to the *gridftp.conf*:

```sh
control_interface 0.0.0.0
```
It forces the server to listen to all ipv4 interfaces.


[]()|[]()
----|----
[Index](https://github.com/EUDAT-Training/B2SAFE-B2STAGE-Training)  | [Next](09-install-B2STAGE.md)

















