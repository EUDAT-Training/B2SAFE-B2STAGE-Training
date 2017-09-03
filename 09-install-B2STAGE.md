# Installing the iRODS-DSI
With the iRODS-DSI all commands executed via the gridFTP protocol will be directly forwarded to iRODS. That means that after the installation you will no longer be able to access the normal filesystem via this protocol. 
A full installation and configuration guide is provided [here](https://github.com/EUDAT-B2STAGE/B2STAGE-GridFTP).

## Necessary system packages
```sh
sudo apt-get install libglobus-common-dev 
sudo apt-get install libglobus-gridftp-server-dev 
sudo apt-get install libglobus-gridmap-callout-error-dev
sudo apt-get install libcurl4-openssl-dev
sudo apt-get install build-essential make cmake git
sudo apt-get install -y cdbs
```

## Necessary iRODS packages and code
```sh
mkdir -p ~/iRODS_DSI
cd ~/iRODS_DSI
wget ftp://ftp.renci.org/pub/irods/releases/4.1.10/ubuntu14/irods-dev-4.1.10-ubuntu14-x86_64.deb
sudo dpkg -i irods-dev-4.1.8-ubuntu14-x86_64.deb
wget ftp://ftp.renci.org/pub/irods/releases/4.1.10/ubuntu14/irods-runtime-4.1.10-ubuntu14-x86_64.deb
sudo dpkg -i irods-runtime-4.1.8-ubuntu14-x86_64.deb
sudo apt-get update
```

```sh
git clone https://github.com/EUDAT-B2STAGE/B2STAGE-GridFTP.git
```

### Installation
```sh
cd B2STAGE-GridFTP
cp setup.sh.template setup.sh
```

Edit the *setup.sh*, minimal setup:

```sh
export GLOBUS_LOCATION="/usr"
export IRODS_PATH="/usr"
export DEST_LIB_DIR="/home/alice/iRODS_DSI"
export DEST_BIN_DIR="/home/alice/iRODS_DSI"
export DEST_ETC_DIR="/home/alice/iRODS_DSI"
```
and install:

```sh
source setup.sh
cmake CMakeLists.txt
make
```

### Configuration
All commands coming from gridFTP entering iRODS will be executed as the same irods user. This userprofile is defined under *root*:

```sh
sudo su -
root@iRODS4:~# mkdir .irods
vim ~/.irods/irods_environment.json
{
"irods_host" : "<fully-qualified-hostname>",
"irods_port" : 1247,
"irods_user_name" : "alice",
"irods_zone_name" : "aliceZone",
"irods_default_resource" : "demoResc"
}
```
Note, that when you copy an irods environment file over or when you create this file with *iinit* the value *irods_default_resource* is not automatically set. However, the iRODS-DSI is dependent on this value and not setting it will cause unpredictable errors.

Do an 
```sh
iinit
```
to create the password-hash that will be used later to automatically authenticate with iRODS.

Add the following to your */etc/gridftp.conf* file:
```sh
# globus-gridftp-server configuration file

# this is a comment

# option names beginning with '$' will be set as environment variables, e.g.
$GLOBUS_ERROR_VERBOSE 1
$GLOBUS_TCP_PORT_RANGE 50000,51000

# port
port 2811

#iRODS connection
$LD_LIBRARY_PATH "$LD_LIBRARY_PATH:/home/alice/iRODS_DSI/"
$irodsConnectAsAdmin "rods"
load_dsi_module iRODS
auth_level 4
```

Restart the gridFTP server:
```sh
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/home/ubuntu/iRODS_DSI/B2STAGE-GridFTP/"
/etc/init.d/globus-gridftp-server restart
```

### Adjusting the grid-mapfile
In the previous step we set up a gridFTP server that uses the unix file system. Thus, in the grid-mapfile the certficate DN was mapped to a unix user. We can reuse the same certficates but need to couple them to the respective iRODS accounts:

```sh
"/O=Grid/OU=GlobusTest/OU=simpleCA-eve-server/OU=Globus Simple CA/CN=ubuntu" rods
```

### Enabling PID resolution on the server
The iRODS-DSI supports accessing data in iRODS with their PID like:
```sh
eve@eve:~$ globus-url-copy -list gsiftp://alice.eudat-sara.vm.surfsara.nl/846/70c2995c-2d80-11e6-acfc-04040a64008f/
gsiftp://alice.eudat-sara.vm.surfsara.nl/846/70c2995c-2d80-11e6-acfc-04040a64008f/
    file.txt
```
To enable this feature simply add

```sh
#Resolution of PIDs
$pidHandleServer "http://<handle-server>:<port>/api/handles"
$pidHandleServer "http://hdl.handle.net/api/handles"
```
E.g. for a local Handle server add: 
```sh
$pidHandleServer "http://epic3.storage.surfsara.nl:8001/api/handles"
```
to your *gridftp.conf* and restart the gridFTP server.

### Combining B2SAFE and B2STAGE
PIDs are usually created with the respective rule in B2SAFE. B2STAGE is capable of resolving the PIDs to their data in iRODS. To this end the URL field in the PID entry needs to contain a path which B2STAGE links to its iRODS instance.
The URL field is set by B2SAFE automatically employing the *serverID* from B2SAFE. This variable needs to match exactly the hostname you used to request the host certficate during the [gridFTP server installation](https://github.com/EUDAT-Training/B2SAFE-B2STAGE-Training/blob/master/08-install-gridFTP-server.md).

Alternatively, you can also adopt the variable *serverID* in */opt/eudat/b2safe/rulebase/local.re* - getEpicApiParameters to make both server names match.

### Testing the iRODS-DSI
As a user initialise a proxy

```sh
grid-proxy-init
```

List data in the user's iRODS home collection:
- Listing
```sh
alice@irods4:~$ globus-url-copy -list gsiftp://alice.eudat-sara.vm.surfsara.nl/aliceZone/home/alice/
```

The output should look like this:
```
gsiftp://irods4-alicetest.eudat-sara.vm.surfsara.nl/aliceZone/home/alice/
put1.txt
put2.txt
test.txt
DataCollection/
DataTrunk/
testData/
```

### Debug information
#### Invalid Authentication
When the gridFTP server is run under root and you restart the server using *sudo* under an admin user you will encounter this error after restarting the gridFTP server:
```
alice@ubuntu:~/iRODS_DSI/B2STAGE-GridFTP$ globus-url-copy -list gsiftp://alice.eudat-sara.vm.surfsara.nl/aliceZone/home/alice/
gsiftp://aliceZone.eudat-sara.vm.surfsara.nl/aliceZone/home/


error: globus_ftp_client: the server responded with an error
530 530-Login incorrect. : /home/alice/iRODS_DSI/B2STAGE-GridFTP/DSI/globus_gridftp_server_iRODS.c:globus_l_gfs_iRODS_make_error:579:
530-iRODS DSI. Error: 'clientLogin' failed.. CAT_INVALID_AUTHENTICATION: , status: -826000.
530-
530 End.
```

You need to restart the gridFTP server directly under root and not via sudo.

#### PID resolving
The PID resolving is tricky. The URL entry of the PID needs to be minted in a way that the gridFTP server knows that the data is lying in its iRODS instance. 

When the PID is not minted correctly or the data you try to fetch is indeed lying on a different iRODS server, you will encounter this error:

```sh
error: globus_ftp_client: the server responded with an error
500 500-Command failed. : /home/cloud-user/iRODS_DSI/B2STAGE-GridFTP/DSI/globus_gridftp_server_iRODS.c:globus_l_gfs_iRODS_stat:844:
500-iRODS DSI: the Handle Server 'http://epic3.storage.surfsara.nl:8001/api/handles' returnd the URL 'irods://di4r2016-3.novalocal/aliceZone/home/alice/testfile.txt' which is not managed by this GridFTP server which is connected through the iRODS DSI to: 192.168.17.53
500-
500 End.
```
The *DSI* tells you where the PID resolves to (*irods://di4r2016-3.novalocal:1247/aliceZone/home/alice/testfile.txt*) and gives you the name of the iRODS instance it knows (*192.168.17.53*). So in the case above the URL field of the PID should state

```
irods://192.168.17.53:1247/aliceZone/home/alice/testfile.txt
```

To make sure that B2SAFE mints the PIDs correctly,  you need to adjust the serverID in */opt/eudat/b2safe/rulebase/local.re*.

[]()|[]()|[]()
----|----|----
[Previous](08-install-gridFTP-server.md)|[Index](https://github.com/EUDAT-Training/B2SAFE-B2STAGE-Training)  | [Next](10-using-B2STAGE.md)
