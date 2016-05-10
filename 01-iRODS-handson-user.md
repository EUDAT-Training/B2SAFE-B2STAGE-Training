# iRODS for users
This lecture illustrates what iRODS is and how you can manage data with iRODS as a user. To this end we wil make use of the icammands.


## Prerequisites
- A user account on an iRODS 4.1.X system
- icommands client, [Installation](http://irods.org/download/)

## Outline
The whole tutorial will guide you through the workflow indicated in the figure below. This part is about **Step 1** ingesting data and administering data in iRODS via the icommands. You will have the role as an iRODS user.
All commands shown in this part are either icommands or shell commands.


### Connecting to the iRODS server
First we connect to an iRODS server and authenticate as iRODS user. The user account has to be created by the iRODS admin beforehand.

```sh
iinit
```
When you connect for the first time, you will receive this answer:

```sh 
One or more fields in your iRODS environment file   (irods_environment.json) are
missing; please enter them.
```
Usually iinit uses the irods_environment.json to retrieve information to which iRODS instance to connect to and which user to use. If the file is incomplete or has not yet been generated you will have to provide this information:

```sh
Enter the host name (DNS) of the server to connect to:  <ip adrdress or fully qualified hostname>
Enter the port number: 1247 
Enter your irods user name: <irodsuser>
Enter your irods zone: <zonename>
```
The port numer is standard 1247. The zone name, username and password will be provided by the iRODS admin.

### Some iRODS concepts
**iRODS zone**: always contains exsctly one so-called iCAT catalogue, which is a database containing user information, the mapping from physical storage to iRODS logical path for data and hosts metadata attached to data.

**Resources**: Software or Hardware system that stores data. The iRODS system abstracts from the hardware and software so that you, as a user, can put data into certain resources without specific knowledge on the protocols to use.

**iRODS collections**: As a user you have access to a collection, just as a home dorectory in a linux system. In this collection you can create subcollections and store data. You can retrieve and store data and collections by using the iRODS (virtual) path. The iCAT catalogue will take care of the mapping to the actual physical path.

### The iRODS environment
With the following command you can retrieve some information on the iRODS system you are working on:
```sh
ienv
```
You will see an answer of the system similar to the one below:
```sh
NOTICE: Release Version = rods4.1.6, API Version = d
NOTICE: irods_session_environment_file - /home/irodsadmin/.irods/irods_environment.json.2440
NOTICE: irods_user_name - irods
…
NOTICE: irods_zone_name – aliceZone
NOTICE: created irodsHome=/aliceZone/home/irods
NOTICE: created irodsCwd=/aliceZone/home/irods
```

**Some useful commands for session management**
------------|---------
iinit       | Log on
iexit       | Log off
ienv        | Client settings
ihelp       | List of icommands
ipasswd     | Change iRODS password
iuserinfo   | User info
ierror      | Information on error code

To see which physical resources are attached to the iRODS instance and what their logical names are, you can use:
```sh
ilsresc –l 
```
which will yield:
```sh
resource name: demoResc
id: 9101
zone: aliceZone
type: unixfilesystem
class: cache
location: iRODS4.alice
vault: /irodsVault
```
This tells us that there is only one resource defined which is of type *unix file system*. The value after *vault* tells us where our data will be stored physically on the iRODS server. *location* tells us the hostaname of the iRODS server. Check with:

```sh
hostname
```
on your shell.

### Working in the iRODS environment
The most important icommand will be:
```sh
ihelp
```
This will print out all commands the client knows.

Let's have a look at our current iRODS working directory and list the content.
```sh
ils
```
Since we have not put any data yet, you will receive this as an answer from the system:
```sh
/alicetestZone/home/alice:
```
Let's create a subcollection. You may already know the UNIX command *mkdir*. iRODS uses a similar command and syntax for creating collections:
```sh
imkdir testData
```
We can remove a file and collection by
```sh
irm -r testData
```

### Adding data to iRODS and retrieveing data from iRODS
Let's create a file and put it into iRODS

```sh
echo "test content" > put1.txt
```
This creates a test file *put1.txt* with *test content*.
You can store into iRODS and by this transferring it to the iRODS server (if you are working on a remote computer).

```sh
iput put1.txt
```
We can list the content again, when using the option *-L* we can also see where iRODS stored the file physically on the server.

```sh
ils -L
```

iRODS will give us the user, which resource it is stored on and as last information the physical path on the iRODS server.
```sh
/aliceZone/home/alice:
  alice             0 demoResc           13 2016-02-19.13:35 & put1.txt generic    /irodsVault/home/alice/put1.txt
```

*iput* comes with some useful options.
```sh
iput -K put1.txt
```
Will store a checksum with your file. If you now execute this command iRODS will complain that the file already exists.
With the *-f* option you can force iRODS to overwrite existing data.
```sh
iput -K -f put1.txt
```
If you now list the collection content again with the *-L* option you caninspect the md5 checksum.
You can also specify which subcollection and which resource iRODS should use to store the data.

```sh
iput -K -f put1.txt -R demoResc testData
```
will store the data physically on demoRes and use /alicetestZone/home/alice/testData as logical path.

To retrieve data from iRODS you can use
```sh
iget -K -P -f put1.txt
```
The option *-K* tells iRODS to verify the chacksum on the fly.
To list all options for the command use
```sh
iput -h
```

### Removing data
To remove our put1.txt from iRODS use

```sh
irm put1.txt
```
Let's inspect what happens.
If we list the content of our current working collection, we will not find the file, so it seems to be deleted. However, inspecting the *trash* folder, shows that only the file's physical and logical path was changed. This is what we call a soft delete.
```sh
ils -L /aliceZone/trash/home/alice
```
```sh
/aliceZone/trash/home/alice:
  alice             0 demoResc           13 2016-02-19.13:52 & put1.txt
    d6eb32081c822ed572b70567826d9d9d    generic    /irodsVault/trash/home/alice/put1.txt
  C- /aliceZone/trash/home/alice/Data
```
That means you can restore the file with the follwing commands.
```sh
imv /eveZone/trash/home/eve/testData/put1.txt /eveZone/home/eve/testData
```
*imv* can be used to move data and subcollections and to rename them.
```sh
imv /eveZone/home/eve/testData/put1.txt /eveZone/home/eve/testData/put2.txt
```

To remove the file completly from the system, you need to execute 
```sh
irmtrash
```
This is called a hard delete. Now the file is removed from the system and from the iCAT catalogue.

### Accession control
With the option *-A* we can list the accession control list of files and collections.
```sh 
ils -A
```
```
    /aliceZone/home/alice:
            ACL - alice#aliceZone:own   
            Inheritance – Disabled
```
This tells us that /home/alice is only visible by the user *alice* and the irodsadmin, who has access to all data by default.

Let's create a subcollection, out some data into it and open the collection

```sh
imkdir DataCollection
ichmod inherit DataCollection
ichmod ichmod read bobe DataCollection
```
With *ichmod inherit* we assure that all data and subcollections in *DataCollection* will inherit their ACL from the parent collection. After that we grant read-access to another user in the iRODS system.
Check the ACL settings of the collection.
```sh
ils -A DataCollection
```
Now we put some data into the collection.
```sh
iput -K put1.txt DataCollection
```
put1.txt ingertited the ACLs from its parent collection.
Note that when you change the ACLs of the parent collection, the ACLs of all files and subcollections are not automatically updated!

Our user bob can now list the collection and read put1.txt.
```sh
bob@irods4:~$ ils /aliceZone/home/alice/DataCollection
/aliceZone/home/alice/DataCollection:
  put1.txt
```
**Important**: When giving access to files and subcollections, the parent collection needs also to be read or writable.
Copy put1.txt to your home collection and give access to another user.

```sh
icp -f DataCollection/put1.txt put1.txt
ichmod read bob put1.txt
```
If user *bob* now tries to list or retrieve put1.txt in our home collection he will receive the follwing error, although the ACLs on the file itself have been set correctly.

```sh
ERROR: lsUtil: srcPath /aliceZone/home/alice/put1.txt does not exist or user lacks access permission
```
This is due to the fact, that *bob* has no read rights on the parent collection /aliceZone/home/

### Annotating data and queries
iRODS provides the user with the possibility to create **A**ttribute **V**alue **U**nit triplets and store them with the data. The triplets are stored in the iCAT catalogue. By that you can add extra information on the data and collections.

We can store information e.g. on the creation data of a file
```sh
imeta add -d put1.txt "Date" "Nov 2015"
```
Here we created an attribute with the name *Date* and gave it the value *Nov 2015*. The unit section is left empty.

We can also annotate a collection
```sh
imeta add -C DataCollection "Type" "Collection"
```
To list AVUs stored for a file use:
```sh
imeta ls -d put1.txt
```
The imeta command also allows us to define simple queries.
```sh
imeta qu -d Date = "Nov 2015"
```
For more sophisticated sql-like queries we can use

```sh
iquest "select sum(DATA_SIZE), COLL_NAME where COLL_NAME like '/alicetestZone/home/alice/Data%'"
```
This command sums the sizes of all data in each collection starting with *Data*. The command *iquest* already knows some keywords specific to the iRODS environment. You can list them with

```sh
iquest attrs
```



