# Installation of B2SAFE 2
This document describes how to install B2SAFE 2 with iRODS4.1 on a Ubunutu 14.04 system.

## Environment
Ubuntu 14.04 server, iRODS 4.1 with postgresql 9.3
You will also need a handle prefix and the respective credentials to configure B2SAFE.

##Prerequisites
For a comprehensive documentation please refer to https://github.com/EUDAT-B2SAFE/B2SAFE-core/wiki.

Install git:
```sh
sudo apt-get install git
```

### 1. Clone code and create packages
- Clone the github repository of B2SAFE and create the debian package
```sh
git clone https://github.com/EUDAT-B2SAFE/B2SAFE-core
cd ~/B2SAFE-core/packaging
./create_deb_package.sh
```
- Install the created package as *root*
```sh
dpkg -i /home/alice/debbuild/irods-eudat-b2safe_3.1-0.deb
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

### 3. Python dependencies
- Check dependencies
```sh
cd /opt/eudat/b2safe/cmd
./authZmanager.py -h
./epicclient.py --help
./logmanager.py -h
./messageManager.py -h
./metadataManager.py -h
```

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

### 4. Tests
- B2SAFE installation
```sh
cd ~/B2SAFE-core/rules
irule -vF eudatGetV.r
```
 should return
```sh
*version = 3.1-0
```

- Generating PIDs
* Create a test collection
```sh
mkdir testData
#save some file with content in testData
vim testData/testfile.txt
# ingest collection into iRODS
iput -r testData/
ils
```
* Edit B2SAFE-core/rules/eudatPidsColl.r, adjust coll_path with /\<irodszone\>/home/\<user\>/testData
```sh
irule -vF B2SAFE-core/rules/eudatPidsColl.r
```
* Output should look like this
```sh
*newPID = <prefix>/<somecrypticstring>
``` 
