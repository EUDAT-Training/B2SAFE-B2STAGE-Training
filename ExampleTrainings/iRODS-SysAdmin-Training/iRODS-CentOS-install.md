# iRODS installation for CentOS
Author: Matthew Saum (SURFsara)

Editor: Christine Staiger (SURFsara)

## Update the system
```sh
sudo yum update ; sudo yum upgrade
```

## Set local host name
Add your new hostname to the line starting with 127.0.0.1 and add another line mapping the IP address to the new hostname.
```sh
sudo vi /etc/hosts
```

Example:

```sh
127.0.0.1     localhost localhost.localdomain  irods-centos
::1           localhost localhost.localdomain localhost6 localhost6.localdomain6
you.rIP.ad.dr irods-centos
```

Set the new hostname in the network configuration of your machine

```sh
sudo vi /etc/sysconfig/network 
```

Example:

```sh
# Created by anaconda
HOSTNAME=irods-centos
NOZEROCONF=yes
NETWORKING=yes
```
Then set the  hostname

```sh
sudo hostname -b irods-centos
```

## Firewall configuration
We need to open the normal ssh port and some extra ports for iRODS and postgresql:

```sh
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp -m tcp --dport 1247 -j ACCEPT
sudo iptables -A INPUT -p tcp -m tcp --dport 20000:20199 -j ACCEPT
sudo iptables -A INPUT -p tcp -m tcp --dport 20157 -j ACCEPT
sudo iptables -A INPUT -p icmp -j ACCEPT
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
```
Save the firewall configuration and restart the server:

```sh
sudo service iptables save  

sudo shutdown -r now 
```   

## Create the iRODS service account
iRODS is not run under root but under a dedicated service account. By default this account is called *irods*.

```sh
sudo adduser irods
sudo usermod -aG wheel irods
sudo passwd irods
```

## Create a folder that serves as iRODS Vault

By default the initial installation of iRODS creates the iRODS vault under
`/var/lib/irods/iRODS/Vault/`. If you would like to use a different location you will need to create the folder and grant the iRODS service account read and write permissions to that folder.

```sh
sudo mkdir /irodsVault/
sudo chown irods:irods /irodsVault/
```

## Install postgrasql
By default iRODS uses a postgresql database as iCAT metadata catalogue.
Install porstgresql:

```sh
sudo yum install postgresql-server


sudo service postgresql initdb
```
Before we start the server we have to manually set the host-based access list for postgresql.

```sh
sudo vi /var/lib/pgsql/data/pg_hba.conf
```

Change
 
```sh
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
```
to

```sh
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
```
and start the service:

```sh
sudo service postgresql start
```
and create the *ICAT* database:

```sh
sudo su - postgres
psql

create database "ICAT";
create user irods with password 'irods';
grant all privileges on database "ICAT" to irods;
\q
exit

```

## Download and install iRODS
Change to your iRODS service account:

```sh
sudo su - irods
```

Download the iRODS server software and the database plugin and install the packages:

```sh
export RENCI='ftp.renci.org/pub/irods/releases'
wget \
ftp://$RENCI/4.1.10/centos7/irods-icat-4.1.10-centos7-x86_64.rpm
wget \
ftp://$RENCI/4.1.10/centos7/irods-database-plugin-postgres-1.10-centos7-x86_64.rpm

sudo yum install irods-icat-4.1.10-centos7-x86_64.rpm
sudo yum install irods-database-plugin-postgres-1.10-centos7-x86_64.rpm
```

## Configure iRODS
Configureation of iRODS is done by running the script:

```sh
sudo /var/lib/irods/packaging/setup_irods.sh

```
You will be asked for information on the zone name, server, etc.
Below we give an example configuration:

```sh
iRODS Zone:                 homeZone
iRODS Port:                 1247
Range (Begin):              20000
Range (End):                20199
Vault Directory:            /irodsVault
zone_key:                   TEMPORARY_zone_key
negotiation_key:            TEMPORARY_32byte_negotiation_key
Control Plane Port:         1248
Control Plane Key:          TEMPORARY__32byte_ctrl_plane_key
Schema Validation Base URI: https://schemas.irods.org/configuration
Administrator Username:     rods
Administrator Password:     Not Shown
```

Configuration of the postgresql 'ICAT' database as iRODS iCAT:

```sh
Database Type:     postgres
Hostname or IP:    127.0.0.1 
Database Port:     5432
Database Name:     ICAT
Database User:     irods
Database Password: Not Shown
```
If your postgresql server lies on the same machine as the iRODS server you can use `127.0.0.1` (the localhost) as address, otherwise provide the fully qualified domain name of the machine with the postgresql server.

## Login to iRODS
To test iRODS we login with the irods admin account *rods*:

```sh
iinit

Enter the host name (DNS) of the server to connect to: 127.0.0.1
Enter the port number: 1247
Enter your irods user name: rods
Enter your irods zone: homeZone
Those values will be added to your environment file (for use by
other iCommands) if the login succeeds.
```

and put some data:

```sh
echo "Hello world!" > test.txt
iput -K test.txt
ils -L test.txt
```

