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
127.0.0.1     localhost localhost.localdomain irods-centos.localdomain
::1           localhost localhost.localdomain localhost6 localhost6.localdomain6
you.rIP.ad.dr irods-centos.localdomain fully.qualified.domain.name
```

Then set the  hostname

```sh
sudo hostnamectl set-hostname irods-centos
```

Check that the servername is set correctly after a reboot

```
sudo reboot
```

```
hostname
```

## Firewall configuration
We need to open the normal ssh port and some extra ports for iRODS and postgresql:

```sh
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp -m tcp --dport 1247 -j ACCEPT
sudo iptables -A INPUT -p tcp -m tcp --dport 1248 -j ACCEPT
sudo iptables -A INPUT -p tcp -m tcp --dport 5432 -j ACCEPT
sudo iptables -A INPUT -p tcp -m tcp --dport 20000:20199 -j ACCEPT
sudo iptables -A INPUT -p icmp -j ACCEPT
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
```
Save the firewall configuration and restart the server:

```sh
sudo service iptables save  
sudo shutdown -r now 
```   

## Install postgresql
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

# IPv6 local connections:
host    all             all             ::1/128                 md5
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


## (Optional) Create the iRODS service account
The installation of iRODS in centos looks for a specific unix user called *irods* under which iRODS should be installed. Usually iRODS creates such a user by default. The service account is called *irods* and the there is an *irods* unix group. The service account will not have a home directory, but all files will be stored in */var/lib/irods*. 
In some cases you might want to create the user manually and couple it to certain authentication methods:

```sh
sudo adduser <service account name>
```
Please use in that case the service account name in the iRODS configuration.

## Install iRODS
We offer instructions for both iRODS 4.1.10 and for iRODS 4.2.1. Please choose the version you would like to install.

In both cases iRODS will be installed under `/var/lib/irods` and `/etc/irods`. Currently, these folders belong to root. If you created an own service account you need to grant permissions:

```sh
sudo chown -R <service account name>:<service account name> /var/lib/irods 
sudo chown -R <service account name>:<service account name> /etc/irods
```




### Download and install iRODS 4.1.10

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

### Download and install iRODS 4.2.1
Please install the following packages with yum:

```
https://packages.irods.org/yum/pool/centos7/x86_64/irods-externals-jansson2.7-0-1.0-1.x86_64.rpm
https://packages.irods.org/yum/pool/centos7/x86_64/irods-externals-libarchive3.1.2-0-1.0-1.x86_64.rpm
https://packages.irods.org/yum/pool/centos7/x86_64/irods-externals-clang-runtime3.8-0-1.0-1.x86_64.rpm
https://packages.irods.org/yum/pool/centos7/x86_64/irods-externals-zeromq4-14.1.3-0-1.0-1.x86_64.rpm
https://packages.irods.org/yum/pool/centos7/x86_64/irods-externals-avro1.7.7-0-1.0-1.x86_64.rpm
https://packages.irods.org/yum/pool/centos7/x86_64/irods-externals-boost1.60.0-0-1.0-1.x86_64.rpm
https://packages.irods.org/yum/pool/centos7/x86_64/irods-runtime-4.2.1-1.x86_64.rpm
https://packages.irods.org/yum/pool/centos7/x86_64/irods-icommands-4.2.1-1.x86_64.rpm
https://packages.irods.org/yum/pool/centos7/x86_64/irods-server-4.2.1-1.x86_64.rpm
https://packages.irods.org/yum/pool/centos7/x86_64/irods-database-plugin-postgres-4.2.1-1.x86_64.rpm
```

### Download and install the latest iRODS version
iRODS 4.2.1 can be installed with the package manager *yum*.
```sh
sudo rpm --import https://packages.irods.org/irods-signing-key.asc
wget -qO - https://packages.irods.org/renci-irods.yum.repo | sudo tee /etc/yum.repos.d/renci-irods.yum.repo
```
Install the packages:
```sh
sudo yum install epel-release
sudo yum install irods-server irods-database-plugin-postgres
```

## Create a folder that serves as iRODS Vault

By default the initial installation of iRODS creates the iRODS vault under
`/var/lib/irods/iRODS/Vault/`. If you would like to use a different location you will need to create the folder and grant the iRODS service account read and write permissions to that folder.

```sh
sudo mkdir /irodsVault/
sudo chown <service account name>:<service account name> /irodsVault/
```

If you are working with the standard service account that is yetto be created in the setup, the first time you want to start iRODS will fail, since the account has no access to the *Vault*. In this case set the permissions after the first run of the conguration script and rerun the script.

## Configure iRODS

### Configure iRODS 4.1.10
Configuration of iRODS is done by running the script:

```sh
sudo /var/lib/irods/packaging/setup_irods.sh

```
If you set a different iRODS service account name you will have to set it here when asked for it. By default the configuration script will use *irods*.
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

### Configure iRODS 4.2.1

Start the configuration script:
```sh
sudo python /var/lib/irods/scripts/setup_irods.py
```

You will be asked for the iRODS service account name. By default this will be *irods*.
If you want to install an iRODS iCAT-enabled server choose option *1. provider*. Option 2 will only install an [iRODS resource server](https://docs.irods.org/4.1.10/manual/installation/).

```sh
iRODS user [irods]: <service account name>
iRODS group [irods]: <service account name>

Database Type: postgres
ODBC Driver:   PostgreSQL
Database Host: localhost
Database Port: 5432
Database Name: ICAT
Database User: irods

iRODS server's zone name [tempZone]: <zoneName>
iRODS server's port [1247]:
iRODS port range (begin) [20000]:
iRODS port range (end) [20199]:
Control Plane port [1248]:
Schema Validation Base URI (or off) [file:///var/lib/irods/configuration_schemas]:
iRODS server's administrator username [rods]: <irodsadmin name>
```


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

## Remarks
Everytime you restart the server, the postgresql database needs to be restarted with:

```sh
sudo service postgresql start
```


