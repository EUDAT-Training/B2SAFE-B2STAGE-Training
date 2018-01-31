# Installation of the EUDAT HTTP API for B2SAFE
This document describes how to create a test environment for the HTTP API for the B2SAFE service. 
The API comes in docker containers. For testing we will also use the dockerised iRODS instance provided by the software package. Hence this part of the training is from a systems point of view independent from the other modules. 

However, we strongly advise to first follow the [iRODS](01-iRODS-handson-user.md) and [B2SAFE installation](03-install-B2SAFE.md) training.

## Prerequisites
You need a virtual machine with either Centos 7.X or Ubuntu 14/16. In this tutorial we will guide you through the installation with CentOS 7.3.
For the general setup ofthe HTTP API you need to install docker-engine v1.13, python 3 and git. For the coupling with B2SAFE you will need to obtain a handle (test)prefix (See also instructions in our module on [B2HANDLE](07c-Working-with-PIDs_B2HANDLE.md))

- Install python 3
 ```sh
 sudo yum install python34-setuptools
 sudo easy_install-3.4 pip
 ```
- Install git
 ```sh
 sudo yum install git
 ```
- Install docker 1.13
 ```sh
 sudo rpm --import "https://sks-keyservers.net/pks/lookup?op=get&search=0xee6d536cf7dc86e2d7d56f59a178ac6c6238f52e"
 sudo yum install -y yum-utils
 sudo yum-config-manager --add-repo https://packages.docker.com/1.13/yum/repo/main/centos/7
 sudo yum makecache fast
 sudo yum install docker-engine
 sudo systemctl enable docker.service
 sudo systemctl start docker.service
 ```
 
## Rapydo and the HTTP API
The HTTP API for B2SAFE is based on a framework called Rapydo. This framework will setup several docker containers which contain the HTTP server, the API and, for testing purposes, also an iRODS instance. Rapydo also provides us with the tools to steer these containers, acessing them and making sure that they can communicate with each other.

We first need to download the HTTP API git repository:
```sh
git clone https://github.com/EUDAT-B2STAGE/http-api.git
cd http-api
```
Now we install the dependencies, start rapydo and check which docker containers have been installed.
```sh
sudo -H data/scripts/prerequisites.sh
sudo rapydo init
sudo rapydo start
sudo rapydo status
```
The init and start can take quite some time to install the containers and their content.
You will see that several containers have been installed:
1. The configuration of the backend container tells you which port to use for accessing the API endpoint, here 8080.
2. icat is the docker container with a local iRODS instance the API endpoint is tied to.
3. postgres is the container with the user database for the HTTP API
4. restclient contains the actual implementation of the API

To start the REST API endpoint do
```sh
sudo rapydo shell backend --command 'restapi launch' &
```

Now you can check from any client machine whether the endpointis up and running:
- Commandline check
 ```
 curl -i <IP or FQDN>:8080/api/status
 ```
- Web browser check: type in <IP or FQDN>:8080/api/status
  
\<IP or FQDN\> is the IP address or fully qualified domain name of your machine. If you are working locally you can also substitute them with 'localhost'.
  
If the command delivers a:
```sh
Server is alive!
```
You have a working API endpoint.

## Swagger UI
To start the Swagger User Interface, an API specification where users can test commands and behaviour interactively execute:
```sh
sudo rapydo interfaces swagger &
```
The endpoint for Swagger is http://\<IP or FQDN\>/?docExpansion=none. Users can access this endpoint by webbrowser.

**NOTE** that Swagger is using *localhost*. That measn that when you installed the HTTP API on a rmote server, the Swagger interface will not find the API secifications. (Status Jan 2018)

## HTTP API and B2SAFE
The HTTP API enables features that are dependent on the B2SAFE module. For example users can inspect metadata created by the B2SAFE service and access data by Persistent identifiers. To this end the iRODS instance behind the HTTP API needs to be enabled with the B2SAFE rulebase and B2SAFE behaviour, e.g. generation of PIDs, needs to be enabled by iRODS event hooks.

**NOTE** The HTTP API does not allow you to specifically execute B2SAFE behaviour like data replication or PID creation. 
To show the connection between the HTTP API and B2SAFE, we will install the B2SAFE rulebase and configure an event hook for PID creation and replication. Our data policy is: Every data object and collection under `/tempZone/home/\<user\>/b2safe` will be replicated to `/tempZone/home/\<user\>/b2replication`.

### Prerequisites
- B2HANDLE python library
- Handle prefix, the correspondng certificate and private key

Go to the docker image that runs the iRODS instance and install the B2HANDLE library
```sh
sudo rapydo shell icat
sudo apt-get install python-pip python-dev build-essential
sudo pip install b2handle
sudo pip install queuelib dweepy 
sudo apt-get install python-lxml python-defusedxml python-httplib2 python-simplejson
```
**All following steps need to be carried out int docker container which runs iRODS.**

### Download and install B2SAFE

If you are not in the right docker container do:
```sh
sudo rapydo shell icat
berods
cd
```
- Check out the latest B2SAFE release
 ```sh
 git clone https://github.com/EUDAT-B2SAFE/B2SAFE-core.git
 cd ~/B2SAFE-core/packaging
 ./create_deb_package.sh
 exit
 ```
- Install the package as *root*:
  ```sh
  dpkg -i /home/irods/debbuild/irods-eudat-b2safe_4.1-0.deb
  ```
- Configure B2SAFE (as user *irods*):
  Before proceeding copy the Handle certificates over to the docer image. You will need a *certificate_only.pem* and the corresponding *privkey.pem*. Make sure the user *irods* has access rights to these two files.
  
  ```sh
  berods
  vi /opt/eudat/b2safe/packaging/install.conf
  ```
Adjust 
  ```sh
  SERVER_ID="irods://<fqdn>:1247"
  HANDLE_SERVER_URL="https://epic5.storage.surfsara.nl:8003"
  PRIVATE_KEY="/home/irods/privkey.pem"
  CERTIFICATE_ONLY="/home/irods/certificate_only.pem"
  PREFIX="<PREFIX>"
  HANDLEOWNER="200:0.NA/$PREFIX"
  REVERSELOOKUP_USERNAME="$PREFIX"
  HTTPS_VERIFY="False"
  MSIFREE_ENABLED=false
  ```
  Configure and install B2SAFE:
  ```sh
  source /etc/irods/service_account.config
  cd /opt/eudat/b2safe/packaging/
  ./install.sh
  ```
### Testing B2HANDLE and the epicclient script

We need to make sure that PIDs can be created. Under the irods account (*berods*) in the icat docker container do:
```sh
/opt/eudat/b2safe/cmd/epicclient2.py \
  os /opt/eudat/b2safe/conf/credentials \
  create www.test.com
```
If you receive an output like this:

```sh
21.T12995/eaffe986-0682-11e8-8ec5-0242ac160003irods@rodserver:~$
```
the PID creation works.

### Installing the b2safe replication event hook
Now we install an event hook such that all data under */\<zonename\>/home/\<user\>/b2safe* is replicated to */\<zonename\>/home/\<user\>/b2replication*. By this data and collections receive a PID and several metadata entries are created upon invocation.

 ```sh
 vim /etc/irods/core.re
 ``` 
 
 Add:
 
 ```c
 acPostProcForPut {
        ON($objPath like "/$rodsZoneClient/home/$userNameClient/b2safe/*"){
                writeLine("serverLog","DEBUG replication to
                /$rodsZoneClient/home/$userNameClient/b2replication");
                EUDATReplication("/$rodsZoneClient/home/$userNameClient/b2safe",
                "/$rodsZoneClient/home/$userNameClient/b2replication",
                "true", "true", *response);
        }
 }
 ```
 to the file.
 **NOTE** that PID creation for replicas does not work for the moment (Jan 2018)
 
 ### Eventhook for PID creation only
 This event hook will create PIDs and checksums for all data objects (not collections) under the collection *PID*.
 
  ```c
 acPostProcForPut {
        ON($objPath like "/$rodsZoneClient/home/$userNameClient/PID/*"){
                writeLine("serverLog","DEBUG PID creation");
		msiSysChksumDataObj;
		EUDATCreatePID("None", $objPath, "None", "None", "false", *newPID);
        }
 }
 ```

### Testing the event hooks
We are going to test the event hooks first through triggering them with the icommands.
- Create the collections which trigger the event hook
 ```sh
 imkdir b2safe
 imkdir PID
 ```
- Create a test collection with one file
 ```sh
 mkdir myColl
 echo "my test file" > myColl/test.txt
 ```
- Now upload the collection to b2safe or PID
 ```sh
 iput -r myColl PID
 iput -r myColl b2safe
 ```
Because of the synchronous PID creation both commands can take some time.

Now we are ready to test the HTTP API and trigger PID creation and relpication by uploading data to the correct collections to iRODS [see next chapter](11b-HTTP-API-handson-user.md).


 
 


