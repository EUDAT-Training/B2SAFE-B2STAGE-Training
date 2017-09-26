# Installation of B2SAFE
This document describes how to install the latest version of B2SAFE with iRODS4.1.X on a Ubunutu 14.04 system.

## Environment
Ubuntu 14.04 server, iRODS 4.1 with postgresql 9.3.
You will need to have *root* rights on an iRODS server, a handle prefix and the respective credentials to configure B2SAFE.
To test B2SAFE in a federation you will also need to setup a federation between this iRODS server and another one.

## Prerequisites
For a comprehensive documentation please refer to https://github.com/EUDAT-B2SAFE/B2SAFE-core/wiki.

- Install git:
 ```sh
 sudo apt-get install git
 ```

- Obtain a (test) prefix to create PIDs. 
 1. For a prefix for the **Handle server v7** you will be provided with a prefix and a password. (legacy)
 2. When working with **Handle server v8** you will be provided with certficates (private key and certificate) and a password for the reverse lookup servelet. You will also need to install the B2HANDLE library (see section below).
 
### 1. Clone code and create packages
- Clone the github repository of B2SAFE and create the debian package

 ```sh
 git clone https://github.com/EUDAT-B2SAFE/B2SAFE-core
 cd ~/B2SAFE-core/packaging
 ./create_deb_package.sh
 ```
- PID configuration with *epicclient.py* (legacy)
 If you do not want to add the trusted CA of the epic server to your trusted CAs you need to edit the B2SAFE-core/cmd/epicclient.py:

 ```py
 self.http = httplib2.Http(disable_ssl_certificate_validation=True)
 ```

- PID configuration with epicclient2.py
 The SSL verfication is given as a parameter in the *credentials* file (see below CRED_FILE_PATH). Set this flag to 'false'.

- Install the created package as *root*
 ```sh
 sudo dpkg -i /home/alice/debbuild/irods-eudat-b2safe_3.1-1.deb
 ```
### 2. Configure B2SAFE

```sh
The package b2safe has been installed in /opt/eudat/b2safe.
To install/configure it in iRODS do following as the user who runs iRODS :

# update install.conf with correct parameters with your favorite editor
sudo vi /opt/eudat/b2safe/packaging/install.conf

# install/configure it as the user who runs iRODS
source /etc/irods/service_account.config
sudo su - $IRODS_SERVICE_ACCOUNT_NAME -s "/bin/bash" -c "cd /opt/eudat/b2safe/packaging/ ; ./install.sh"
```
We need to pass some parameters that B2SAFE will use to craete the credentials json file to connect to the Handle server to create PIDs

For working with **Handle v8, the B2HANDLE library and epicclient2.py** you need to fill in the following:
- PRIVATE_KEY: Path to the respective pem-file
- CERTIFICATE_ONLY: Path to the respective pem-file (make sure the linux user *irods* has access to these files)
- REVERSELOOKUP_USERNAME: Usually your prefix
- HTTPS_VERIFY: If you did not install the handle servers certificates, set this variable to "False".

The resulting config file should look like this:
```sh
HANDLE_SERVER_URL="https://epic4.storage.surfsara.nl:8007"
PRIVATE_KEY=/home/ubuntu/HandleTestPre/308_21.T12995_USER01_privkey.pem
CERTIFICATE_ONLY=/home/ubuntu/HandleTestPre/308_21.T12995_USER01_certificate_only.pem
PREFIX="21.T12995"
HANDLEOWNER="200:0.NA/21.T12995"
REVERSELOOKUP_USERNAME="21.T12995"
#REVERSELOOKUP_PASSWORD="" <-- the script will ask for the password upon execution
HTTPS_VERIFY="False"
```
If you added the trusted CA of the epic server to your trusted CAs you can set *HTTPS_VERIFY="True"*

After running the configuration script the resulting json file */opt/eudat/b2safe/conf/credentials* should look like this:
```sh
{
    "handle_server_url": "https://epic4.storage.surfsara.nl:8007",
    "private_key": "/<path>/<to>/308_21.T12995_USER01_privkey.pem",
    "certificate_only": "/<path>/<to>/308_21.T12995_USER01_certificate_only.pem",
    "prefix": "21.T12995",
    "handleowner": "200:0.NA/21.T12995",
    "reverselookup_username": "21.T12995",
    "reverselookup_password": "XXX",
    "HTTPS_verify": "False"
}
```
For a testing server you might want to set *AUTHZ_ENABLED* and *MSIFREE_ENABLED* to false.

By default B2SAFE uses the python script *epicclient.py* which does not make use of the B2HANDLE python library. Copy *epicclient2.py* to *epicclient.py*:
 
 ```sh
 cp /opt/eudat/b2safe/cmd/epicclient.py /opt/eudat/b2safe/cmd/epicclient.py_backup
 cp /opt/eudat/b2safe/cmd/epicclient2.py /opt/eudat/b2safe/cmd/epicclient.py
 ```

#### B2SAFE with Handle v7 and the epicclient.py
**Legacy documentation**

When working with the **epicclient.py** you need to fill in the following parameters:
- SERVER_ID: The fully qualified name of your server or IP address. Note, that when you also plan to install B2STAGE, the server ID has to match the known hostname of your gridFTP instance.
- BASE_URI: The handle server that hosts your handle prefix. Here we use *https://epic3.storage.surfsara.nl/v2_test/handles/*
- USERNAME: This is your handle prefix
- PREFIX: This is also your handle prefix
- CRED_FILE_PATH: The json credentials file to connect to the Handle server

The resulting json file after the configuration should look like this:
```sh
{
    "handle_server_url": "https://epic3.storage.surfsara.nl:8001",
    "prefix": "841",
    "handleowner": "200:0.NA/841",
    "reverse_username": "841",
    "reverse_password": "XXX",
    "HTTPS_verify": "False",
    "overwrite": "True" 
}
```

### 3. Python dependencies
- Check dependencies

 ```sh
 cd /opt/eudat/b2safe/cmd
 ./authZmanager.py -h
 ./logmanager.py -h
 ./messageManager.py -h
 ./metadataManager.py -h
 ```
 With
 ```sh
 ./epicclient.py --help
 ```
 or
 ```sh
 ./epicclient2.py --help
 ```
 you can check whether all additional modules are available.

- Known dependencies

 ```sh
 sudo apt-get install python-pip
 sudo pip install queuelib
 sudo pip install dweepy

 sudo apt-get install python-lxml
 sudo apt-get install python-defusedxml
 sudo apt-get install python-httplib2

 sudo apt-get install python-simplejson
 ```
- B2HANDLE python library

 Install as user with sudo rights
 
 ```sh
 git clone https://github.com/EUDAT-B2SAFE/B2HANDLE
 cd B2HANDLE/
 python setup.py bdist_egg
 cd dist/
 sudo easy_install b2handle-1.0.3-py2.7.egg
 ```

### 4. Tests
#### B2SAFE installation
```sh
iinit
cd ~/B2SAFE-core/rules
irule -vF eudatGetV.r
```
 should return
```sh
*version = 4.X-X
```

#### Generating PIDs
- Test the epicclient.py:
 ```sh
 sudo su - irods
 /opt/eudat/b2safe/cmd/epicclient.py os /opt/eudat/b2safe/conf/credentials create www.test.com
 ```
- Test the epicclient2.py:
 
 ```sh
 sudo su - irods
 /opt/eudat/b2safe/cmd/epicclient2.py os /opt/eudat/b2safe/conf/credentials create www.test.com
 ```
#### B2SAFE tests
In the folder *B2SAFE-core/rules* you will find some test rules that
- Create some PIDs with your installed prefix and deletes them again:
 ```sh
 ubuntu@ubuntu:~$ irule -F rules/eudatCreatePid.r
 userNameClient: rods
 rodsZoneClient: aliceZone
 Object /aliceZone/home/rods/test_data.txt written with success!
 Object contents:
 Hello World!
 The Object /aliceZone/home/rods/test_data.txt has PID = 21.T12995/4412d58a-0cd1-11e7-bdac-040091643b25
 PID 21.T12995/4412d58a-0cd1-11e7-bdac-040091643b25 removed
 Object /aliceZone/home/rods/test_data.txt removed
 ```
- Create a collection with a file, attache some PIDs, replicate the collection to another colletion on the same iRODS server
 ```sh
 ubuntu@ubuntu:~$ irule -F rules/eudatRepl_coll.r
userNameClient: rods
rodsZoneClient: aliceZone
Created collection /aliceZone/home/rods/t_coll
Object /aliceZone/home/rods/t_coll/test_data.txt written with success!
Object contents:
Hello World!
The Collection /aliceZone/home/rods/t_coll has PID = 21.T12995/7dee7516-0cd1-11e7-ae28-040091643b25

Collection /aliceZone/home/rods/t_coll replicated to Collection /aliceZone/home/rods/t_coll2!
The content of the replicated collection is: /aliceZone/home/rods/t_coll2/test_data.txt
The content of the replicated object /aliceZone/home/rods/t_coll2/test_data.txt is:
Hello World!

PIDs for data:
The Original /aliceZone/home/rods/t_coll/test_data.txt has PID = 21.T12995/7ec3c5c2-0cd1-11e7-b1d1-040091643b25
The Replica /aliceZone/home/rods/t_coll2/test_data.txt has PID = 21.T12995/7f3ddf88-0cd1-11e7-8084-040091643b25
Remove replicated data object
PID 21.T12995/7f3ddf88-0cd1-11e7-8084-040091643b25 removed
Replicated object removed

PIDs for collections:
The Original /aliceZone/home/rods/t_coll has PID = 21.T12995/7dee7516-0cd1-11e7-ae28-040091643b25
The Replica /aliceZone/home/rods/t_coll2 has PID = 21.T12995/7e75fe32-0cd1-11e7-84e2-040091643b25
Remove replica Collection and PID.
PID 21.T12995/7e75fe32-0cd1-11e7-84e2-040091643b25 removed
Replicated collection removed

Remove original data and their PIDs
PID 21.T12995/7ec3c5c2-0cd1-11e7-b1d1-040091643b25 removed
Object /aliceZone/home/rods/t_coll/test_data.txt removed
PID 21.T12995/7dee7516-0cd1-11e7-ae28-040091643b25 removed
Removed collection /aliceZone/home/rods/t_coll
 ```
 
### Exercise
(For this exercise you will first need to follow the part on **iRODS rule writing**.)

Alter the test rules in *rules* such that a real folder is assigned with PIDs and replicated to another folder on that iRODS instance. 
Do not remove files, folders and PIDs and verify the correct linking in the iCAT metadata catalogue (*imeta ls*) and the handle system (hdl.handle.org/\<PID\>?noredirect)
 
[]()|[]()|[]()
----|----|----
[Previous](02-iRODS-handson-admin.md)|[Index](https://github.com/EUDAT-Training/B2SAFE-B2STAGE-Training)  | [Next](04-iRODS_federations_configuration.md)
