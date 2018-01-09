# iRODS advanced User Training

- Recap on icommands
- iRODS storage resources and implicit data policies
- iRODS federations
- iRODS rule language and write your own backup data policy

## Recap icommands (15 min)

Command 	| Meaning
---------|--------
iinit		| Login
ienv		| iRODS environment
iuserinfo	| User attributes
**ihelp**		| List of all commands
**\<command\> -h** | Help
**Up- and down load**	|
iput	[-K -r -f -R \<resc\>]	| Upload data, create checksum, recursively, overwrite, specify resource
iget [-K -r -f]	| Check checksum, recursively, overwrite
**Data organisation** |
ils [-L -A -l] | List collection [Long format, Accessions, less long format]
imkdir		| Create collection
icd			| Change current working collection
**Replica handling** |
irepl [-R \<resc\> -a -r]| Replicate btstreamn to another resource
itrim [-n -N -r]| Trim the number of replicas
**System Metadata** | 
ilsresc [-z \<zone name\> -l] | List all storage resource
**Synchronisation** |
irsync [-r -l] | Synchronise data between client and iRODS zone or between iRODS zones
**iRODS rules** |
irule [-F -v]   | Execute a rule
**Metadata** 		|
imeta add [-d -C] Name AttName AttValue [AttUnits]	| Create metadata [file, collection]
imeta ls [-d -C]	| List metadata [file, collection]
iquest		| Find data by query on metadata
iquest attrs	| List of attributes to query 

**Some predefined attributes for iquest:**

USER\_ID, USER\_NAME, RESC\_ID, RESC\_NAME, RESC\_TYPE\_NAME, RESC\_CHILDREN, RESC\_PARENT, DATA\_NAME, DATA\_REPL\_NUM, DATA\_SIZE, DATA\_RESC\_NAME, DATA\_PATH, DATA\_OWNER\_NAME, DATA\_CHECKSUM, COLL\_ID, COLL\_NAME, COLL\_PARENT\_NAME, COLL\_OWNER,\_NAME META\_DATA\_ATTR\_NAME, META\_DATA\_ATTR\_VALUE, META\_DATA\_ATTR\_UNITS, META\_DATA\_ATTR\_ID, META\_COLL\_ATTR\_NAME, META\_COLL\_ATTR\_VALUE, META\_COLL\_ATTR\_UNITS, META\_COLL\_ATTR\_ID, META\_COLL\_CREATE\_TIME, META\_COLL\_MODIFY\_TIME, META\_NAMESPACE\_COLL, META\_RESC\_ATTR\_NAME, META\_RESC\_ATTR\_VALUE, META\_RESC\_ATTR\_UNITS

**Example query**:

```
iquest "select COLL_NAME, DATA_NAME where \
META_DATA_ATTR_NAME like 'author' and META_DATA_ATTR_VALUE like 'Alice'"
```

Remember: '%' is a wildcard in iquest.

### Login
You login to iRODS with the command

```
iinit
```
You will be asked for the iRODS server you would like to connect to the port (standard 1247), the zone name of the iRODS server, your iRODS user name and password.

**NOTE: Adjusting the default checksum (needed for exercises with checksum checking)**

Since iRODS 4.2.0 users need to adjust the default hash scheme for checksums if they want to use MD5 instead of SHA256.

Open the environment file:

```sh
vi .irods/irods_environment.json
```

And insert at the top:

```sh
    "irods_default_hash_scheme": "MD5",
```

### Basic commands
First we will have a look at some very basic commands to move through the logical namespace in iRPDS. The basic commands in iRODS are very similar to bash/shell commands.
You can browse through your coollections with:

```
ils
```
And you can create new collections with:

```
imkdir lewiscarroll
```

### Uploading data to iRODS
We can put a single file into our home-collection or a designated collection.

```
iput -K aliceInWonderland-DE.txt.utf-8
```
The flag *-K* triggers the calculation and verification of a checksum, in this case an md5 sum or digest of the sha256 checksum. 
Now upload a collection to iRODS:

```
iput -r -K aliceInWonderland lewiscarroll/alice
```
The option *K* triggers the calculation of checksums and verifies them upon upload.

### Logical and physical namespaces in iRODS
The *ils* command gives you an option to extract the physical location of a file:

```
ils -L -r lewiscarroll/alice
```

You will see some output like:

```
/aliceZone/home/irods-user1/lewiscarroll/Alice:
  irods-user1       0 demoResc       109858 2018-01-03.09:43 
  & aliceInWonderland-EN.txt.utf-8 4469a7b948107c7d5bba84b0403cd415   
  generic    
  /var/lib/irods/Vault/home/irods-user1/lewiscarroll/Alice/aliceInWonderland-EN.txt.utf-8
```

We will use this command quite often today to see what happens with files upon replication.
With this command you see where the file is stored on the iRODS server.

- *alice* is the data owner
- *0* is the index of this replica. The number only refers to replicas in one iRODS zone and can be used to automatically trim the number of replicas or create new ones in case one got lost.
- *demoResc* is the resource on which the data is stored. Resources can refer to certain paths on the iRODS server or other storage servers and clusters.
* The next entries are the size of the file, the time of the upload and the file name
* The '&' indicates that the checksum was successfully veryfied; i.e. the checksum on the underlying storage matches the checksum in the iCAT
* The following entry is the checksum
* The last entry is the physical path of the data on the underlying storage, in our case the data lies on the iRODS server.

### Exercise (5 min)

Data can be downloaded from iRODS to your local machine with the command *iget*.
Explore the command *iget* to **store the data in your home directory**. Do **not overwrite** your original data and **verify checksums**!

## iRODS resources (30 minutes)

With the command `ils -L` we explored the link between the iRODS logical namespace and the the physical location of files and folders. The same can be done with resources.

iRODS resources are pieces of a file system, external servers or software in which data can be stored.

You can list all resources you have available with:

```
ilsresc
``` 
You will see the resource tree.

```
demoResc:unixfilesystem
iarchive-centosResource:unixfilesystem
replResc:replication
├── resource1:unixfilesystem
└── resource2:unixfilesystem
robin:roundrobin
├── resource3:unixfilesystem
└── resource4:unixfilesystem
```
There are the storage resources: demoResc, iarchive-centosResource, storage1, storage2, ...

The resources demoResc and iarchive-centosResource can be used directly to store data.
The resources storage1-4 are managed by coordinating resources which implement low-level data policies.

If not further specified all your data will go to 'demoResc'.

You can specify the resource on which your data shall be stored directly with the *iput* command. Let us put some data on the **iarchive-centosResource** resource.

We first create a *test.txt*

```
echo "Some test text" > test.txt
```

```
iput -K -R iarchive-centosResource test.txt testfile-on-iarchive.txt
```

BIG advantage: As a user you do not need to know which storage medium is hidden behind the resource, you simply use the 
icommands to steer your data movements in the backend.

### User-defined replication of data
Once your data is managed by iRODS you can also replicate your data to another predefined resource.
Use *irepl* to replicate the German version of Alice in Wonderland to **iarchive-centosResource**.

```
irepl -R iarchive-centosResource \
lewiscarroll/alice/aliceInWonderland-EN.txt.utf-8
```

```
ils -l lewiscarroll/alice/aliceInWonderland-EN.txt.utf-8
  irods-user1       0 demoResc       109858 2018-01-03.09:43 & 
  	aliceInWonderland-EN.txt.utf-8
  irods-user1       1 iarchive-centosResource       109858 2018-01-03.09:50 & 
  	aliceInWonderland-EN.txt.utf-8
```

The replicas are enumerated. With this number you can specify a replica in the commands *irm* and *itrim*. Let us remove the replica on the demoResc:

```
irm -n 0 lewiscarroll/alice/aliceInWonderland-EN.txt.utf-8
``` 
We still have a copy of the English version in our system, so the logical name still exists:

```
ils lewiscarroll/alice
/aliceZone/home/alice/lewiscarroll/alice:
  aliceInWonderland-EN.txt.utf-8
  aliceInWonderland-IT.txt.utf-8
```

If you do an 

```
irm aliceInWonderland-EN.txt.utf-8
```
all replicas will be removed. 

### Small exercise
1. Replicate a file to two different resources
3. What happens if you want to store data on resource1?
2. Explore `itrim` and trim the number of replicas to 1 (1 original and 1 replica)

### Summary so far

- Child resources cannot be accessed directly
- Replicas are numbered sequentially, new replica always receives the successor of the highest index
- Note the difference between the option -n and -N!
- When trimming replicas to a specific number, the replicas with the highest indexes are removed
- **Watch out**: replicas are not automatically updated (no automatic synchronisation). If you overwrote a replica the '&' sign in the listing will dissappear for all other replicas. You can trigger the update with `irepl -a <irods path>`

### Resources have a life of their own (Low-level data policies)
In iRODS the system admin can group resources and enforce certain data replication or copying policies.

```
iput -K -R robin test.txt testfile-on-rr.txt
ils -l testfile-on-rr.txt
```

### Small exercise
- Where is your data stored physically. 
- Where is your neighbours data stored?
- Upload several files after each other. Where does the data land physically?

## Exercise: Explore the data policy behind replResc (20min)

We have seen that a round robin implements a certain data policy. Which data policy is hidden behind 'replResc'?

1. Consider putting data and trimming data.
2. Where are all the resources located? (`ilsresc -l`)

**Note** As an iRODS user you do not need to know which servers and storage systems are involved. 
You only need an idea about the policies hidden behind grouped resources.

## Exercise: Where is the test file we put on 'iarchive' physically located? (10min)
Hints: 

1. You need to combine information from the iRODS system and the linux system
2. Where are all the resources located? (`ilsresc -l`)
3. How many servers does the iRODS system use?

## iRODS federations (10 min)
iRODS federations are connections between different iRODS servers or - in iRODS terms - *zones*. iRODS federations are setup by the system administrators. They acknowledge users from the federated zone, which allows you as a user to read and write data at the remote/federated iRODS zones.

In our example our users are known and authenticated at *aliceZone*. Each of your accounts has a counter part at the remote zone *bobZone*. 

Let us have a look at how we can access our **remote** home directory.

```
ils /bobZone/home/alice#aliceZone
/bobZone/home/alice#aliceZone:
```

Note that when you are accessing your remote home you have to state at which iRODS zone you are authenticated. This is indicated with the *#aliceZone*.

We can put data directly from our linux account in the remote home:

```
iput -K aliceInWonderland-DE.txt.utf-8 /bobZone/home/alice#aliceZone
```

```
ils /bobZone/home/alice#aliceZone -L
/bobZone/home/alice#aliceZone:
  alice        0 demoResc       187870 2017-03-27.10:20 
  & aliceInWonderland-DE.txt.utf-8
    7bdfc92a31784e0ca738704be4f9d088    generic    
    /irodsVault/home/alice#aliceZone/aliceInWonderland-DE.txt.utf-8
```

Most icommands like *iput*, *iget*, *imeta* and *imkdir* work just like in our home zone, you only need to specify the full iRODS path. I.e. you can  upload data from your client machine and you can work with data within the remote iRODS zone.

Commands to retrieve information from the iCAT like *ilsresc* have an option *-z \<zoneName\>* to access a remote iCAT.

**Exercise** List all resources on bobZone.

However, when working with data between two iRODS zones you will notice that not all operation are allowed.

#### Small exercise (10min)
- Try to use *imv* to move *aliceInWonderland-DE.txt.utf-8* from *aliceZone* to *bobZone*. Can you use *icp*? What could be the reasoning for the different behaviour?
- Download the German version from *bobZone* to your local linux filesystem (store it under a different file name). Which commands can you use?

The command *imv* edits the corresponding entry in the iCAT metadata catalogue at *aliceZone* and moves the data physically to a new location in the *Vault*.

```
ils -L aliceInWonderland-DE.txt.utf-8
imv aliceInWonderland-DE.txt.utf-8 aliceGerman.txt
ils -L aliceGerman.txt
```
*imv* would mean that the metadata entry is at *aliceZone*, while the data is physically stored at *bobZone*.
With *icp* you create a new data object with a metadata entry at its iRODS zone and storage site.

### Cross-zone replication and synchronisation (15min)
As with *gridFTP* and *rsync* iRODS offers a command to synchronise data between either your local unix filesystem or between different iRODS zones.
In contrast to pure iRODS replication with *irepl*, the *irsync* command will create new data objects and collections with -by default- empty metadata.

In the following we will use the remote iRODS zone as a backup server. The iRODS collection *archive* will serve as source collection for the backup.

Upload one of the files in *aliceInWonderland* to your home-collection on **aliceZone**. Do not use the *-K* flag.

```
iput aliceInWonderland/aliceInWonderland-EN.txt.utf-8
```

We can replicate the file to *bobZone*

```
irsync i:/aliceZone/home/alice/aliceInWonderland-EN.txt.utf-8 \
i:/bobZone/home/alice#aliceZone/aliceInWonderland-EN.txt.utf-8
```

Check 

```
ils -L /bobZone/home/alice#aliceZone/aliceInWonderland-DE.txt.utf-8 
```

You will find that both files, the local one and the remote, carry checksums. *irsync* calculates the checksums and uses them to determine whether the file needs to be transferred. 
File transfers with *irsync* are quicker when first calculating the checksum and then transferring them.

#### Small Exercise
Which commands can you use to download the data from the remote iRODS zone to your local unix file system?

### Exercise (15min)
Verify that *irsync* really just updates data when necessary.

1. Create a collection on *aliceZone*, e.g. *archive*
2. Add some files to this collection, e.g. the German version of Alice in Wonderland (use *icp* or *imv*, make sure checksums are calculated) and synchronise to bobZone.
3. Check what needs to be synchronised with *irsync -l* flag. What does this flag do?
4. Add another file to *archive* on *aliceZone*, e.g. the Italian version of Alice in Wonderland.
5. Check what needs to be synchronised and synchronise.
6. Check the synchronisation status. (It can take some time until the iRODS system marks the new files as 'synchronised').

#### Solution

1. Synchronising

 ```
 imkdir archive
 icp -K aliceInWonderland-DE.txt.utf-8 archive
 irsync -r i:archive i:/bobZone/home/alice#aliceZone/archive
 irsync -r -l i:archive i:/bobZone/home/alice#aliceZone/archive
 ```
2. Add new files to *archive*

 ```
 iput aliceInWonderland/aliceInWonderland-IT.txt.utf-8 archive/
 ```
3. Check sync-status
 
 ```
 irsync -r -l i:archive i:/bobZone/home/alice#aliceZone/archive
 /aliceZone/home/alice/archive/aliceInWonderland-IT.txt.utf-8   175251   N
 ```
4. Synchronising and checking sync-status
 
 ```
 irsync -r i:archive i:/bobZone/home/alice#aliceZone/archive
 irsync -r -l i:archive i:/bobZone/home/alice#aliceZone/archive
 ```
 
 If you did not calculate the checksums for the source files, the sync-status needs some time to be updated.

### Metadata for remote data (5min)
We created a nother copy of the *archive* collection at *bobZone* but we lost the link to the data at *aliceZone*.
We will now have a loko at how we can use the iCAT metadat catalogues at *bobZone* and at *aliceZone* to link the data.

Recall, we can create metadata for iRODS data objects and collections on our home iRODS zone like this:

```
imeta add -C archive "SOURCE" "/aliceZone/home/alice/archive"
imeta add -d archive/aliceInWonderland-DE.txt.utf-8 \
"SOURCE" "/aliceZone/home/alice/archive/aliceInWonderland-DE.txt.utf-8"
```

With 

```
imeta ls -C archive
```
and

```
imeta ls -d archive/aliceInWonderland-DE.txt.utf-8
```
we can list all metadata.

We can do exactly the same for the data at the remote site

```
imeta add -C /bobZone/home/alice#aliceZone/archive \
"SOURCE" "/aliceZone/home/alice/archive"
```
We created a new metadata entry in the remote iCAT.

#### Small exercise (5min)

1. Label the files in */bobZone/home/alice#aliceZone/archive* with information on its original source.
2. Introduce anonther metadata field in the original data to link to the replicas. Use the key "REPLICA".

### Retrieving data by metadata (10min)
It is worth mentioning that you are not able to query the iCAT catalogue of a remote zone.

We can retrieve our freshly labeled data at *aliceZone*

```
iquest "select COLL_NAME where META_COLL_ATTR_NAME like 'SOURCE'"
COLL_NAME = /aliceZone/home/alice/archive
```

We only receive the collection which lies in our home zone. Let's try to get the remote collection.

```
iquest "select COLL_NAME where \
META_COLL_ATTR_NAME like 'SOURCE' and COLL_NAME like '%bobZone%'"

CAT_NO_ROWS_FOUND: Nothing was found matching your query
```

Now try the same query with 'iquest -z bobZone':

```
iquest -z bobZone "select COLL_NAME where META_COLL_ATTR_NAME like 'SOURCE'"

Zone is bobZone
COLL_NAME = /bobZone/home/irods-user1#aliceZone/archive
```

**--> no truly federated iCAT search**. You will have to issue separate searches for different iCAT databases.

#### Small exercise (10 min)
Assume your "SOURCE" *archive* collection on aliceZone is corrupted. How would you find out where you can get a copy of that data and how would you restore the data?

#### Solution
```
iquest "select META_COLL_ATTR_VALUE where META_COLL_ATTR_NAME like 'Replica' and COLL_NAME like '%archive%'"
irsync -r -l i:\<answer from iquest\> i:/aliceZone/home/alice/archive
irsync -r i:\<answer from iquest\> i:/aliceZone/home/alice/archive
```

### Summary slides

## iRODS rules (30min)
In the previous parts we did a lot of work manually:

- replicating data to a different zone
- labeling data to keep track of originals and replicas

iRODS offers the possibility to automate data management processes by creating scripts written in the iRODS rule language. We will first inspect the iRODS rule language and then automatise the steps of the previous section and finally schedule the backup process in regular time intervals.

Save the example rule below in a file called *HelloWorld.r*

```
HelloWorld{
	writeLine("stdout", "Hello *name!");
}

INPUT *name="YourName"
OUTPUT ruleExecOut, *name
```

and execute the rule with

```sh
irule -F exampleRules/helloworld.r
```

### Passing arguments, variables and ouput
The rule has an input variable which we did not set in the previous call. The default value for the variable is "YourName".
To customise the function, we could alter the code, or we could pass on the right value for the variable.

```
HelloWorld{
	writeLine("stdout", "Hello *name!");
}

INPUT *name="YourName"
OUTPUT ruleExecOut, *name
```

We can overwrite input parameters by calling the function like this:

```sh
irule -F exampleRules/helloworld.r "*name='Alice'"
```

### Exercise: Passing variables, data types (10min)

```
variables{
	writeLine("stdout", "var1 is *var1!");
	writeLine("stdout", "var2 is *var2!");
}

INPUT *var1=1, *var2="string"
OUTPUT ruleExecOut, *name
```

- Alter the type of the input variables: numbers and simple calculations, booleans (true, false), strings
- How do you only change one of the two variables?

```sh
irule -F exampleRules/variables.r '*var1="Hello"'
```

```sh
irule -F exampleRules/variables.r "*var1='456'" "*var2='true'"
```

```sh
irule -F exampleRules/variables.r '*var1=3+4/7.'
```

```sh
irule -F exampleRules/variables.r "*var1='Hello'" "*var2=Hello"
```

### Global/System variables

iRODS knows predefined global variables that are set by the system and can come in handy. Those variables are addressed by "$" just like in shell scripting.

```sh
$userNameClient
$rodsZoneClient
$objPath

```

With them you can e.g. create the home collection of the active user:

```sh
*home="/$rodsZoneClient/home/$userNameClient"
```

### Looping over collections

iRODS is a data management software. In most cases we would like to loop over data objects and collections.
This is a rule that lists all data objects in a user's 'home' collection.

```c
recursivelist{
    *home="/$rodsZoneClient/home/$userNameClient"
    writeLine("stdout",*home);
    foreach(*row in SELECT COLL_NAME, DATA_NAME WHERE COLL_NAME like '*home%'){
        *coll = *row.COLL_NAME;
        *data = *row.DATA_NAME;
        writeLine("stdout", "*coll/*data");
    }
}

input null
output ruleExecOut
```
```sh
irule -F exampleRules/recursivelist.r
```
The '%' works as wild card, variables are denoted by '*'.

We see that this kind of for-loops use statements similar to the ones in the *iquest* command to retrieve data and collections.

### Exercise (15min)
Write a rule that finds all data objects and all collections that carry the same metadata attribute.
E.g. there is a collection labeled with the attribute 'game' and there are some files carrying the same attribute. Make the attribute a variable.

### Solution framework

```c
queryall{
	foreach(*row in SELECT COLL_NAME, <FILL IN> where <FILL IN> like '*var'){
		*coll = *row.COLL_NAME;
       *value = *row.<FILLIN>;
       writeLine("stdout", "<Some output>");
   	}
   	foreach(*row in SELECT COLL_NAME, <FILL IN>, <FILL_IN> where <FILL IN> like '*var'){
		*coll = *row.COLL_NAME;
		*data = *row.<FILL IN>;
       *value = *row.<FILL IN>;
       writeLine("stdout", "<Some output>");
    }
}

input *var='game'
output ruleExecOut
```

### Solution

```c
queryall{
        foreach(*row in SELECT COLL_NAME, META_COLL_ATTR_VALUE 
        	where META_COLL_ATTR_NAME like '*var'){
        *coll = *row.COLL_NAME;
        *value = *row.META_COLL_ATTR_VALUE;
        writeLine("stdout", "*coll *value");
        }
        foreach(*row in SELECT COLL_NAME, DATA_NAME, META_DATA_ATTR_VALUE 
        	where META_DATA_ATTR_NAME like '*var'){
        *coll = *row.COLL_NAME;
        *data = *row.DATA_NAME;
        *value = *row.META_DATA_ATTR_VALUE;
        writeLine("stdout", "*coll/*data *value");
        }
}

input *var='game'
output ruleExecOut
```

### If-statements and on-statements

```c
conditionalhello{
    if(*name!="Your Name"){
        writeLine("stdout", "Hello *name!");
        }
    else { writeLine("stdout", "Hello world!"); }
}
INPUT *name="Your Name"
OUTPUT ruleExecOut, *name
```
```
irule -F exampleRules/conditionalhello.r "*name='You'"
```

iRODS knows another conditional structure, the on-statement. It can be seen as a *switch statement* in other programming languages.
The same rue above looks like this with on-statements:

```c
hellorule{
    *result = hello(*name);
    writeLine("stdout", "*result");
}

hello(*name){
    on(*name=="Your Name")
        { "Hello world!"; }
}

hello(*name){
    "Hello *name!";
}

INPUT *name="Your Name"
OUTPUT ruleExecOut, *name
```
```
irule -F exampleRules/conditionalhelloon.r
```

The *hello* rules implement single cases of data policies. The *hellorule* puts them together in a sort workflow.
iRODS executes the first *hello* rule that matches the input and leads to some action.

iRODS is a not a real programming language but a rule/policy language. Thus, rules should not be seen as functions but as policies. 

The rules work like a filter. Rules can have the same name and different bodies. The first rule that matches the parameters is executed. Hence, the most general rule (policy) should go to the back.

On-statements enable us to define different policies and update them without breaking other policies. Explore that in the exercise below.

### Exercise (15min)

- Write a rule with different cases (decision between data policies), set the variable "iresource" accordingly:
	- If the data size is large, "iresource" should be "archive"
	- If the data should be highly available, "iresouce" should be "replResc"
	- If the data is classified as sensitive data, "iresource" should always be "storage3"
	- In all other cases the data should go to the "demoResc"
	
- Implement the policies using *if* or *on*.
- Which of the two is more advantagous if you think of what you need to alter when one of the cases (policies) changes?
- Why would you put the cases tested with *on* in different rules (all carrying the same rule name)?

### Solution framework

```c
policydecision{
	# example if
	if(*size=="large"){* resourceName = "archive"}
	else{ ... }
	# example on
	*resourceName = storagepolicy(*size, *privacy, *availability)
	writeLine("stdout", "*resourceName")
}

#example on
storagepolicy(*size, *privacy, *availability){
	on(*availability=="high"){"replResc"}
}

INPUT *size=<FILL IN>, *privacy=<FILL IN>, *availability=<FILL_IN> 
OUTPUT ruleExecOut
```

### Solution with on

```c
policydecision{
        *resourceName = storagepolicy(*size, *privacy, *availability);
        writeLine("stdout", "*resourceName");
}

storagepolicy(*size, *privacy, *availability){
        on(*privacy=="high"){ "storage3"; }
}

storagepolicy(*size, *privacy, *availability){
        on(*availability=="high"){ "replResc"; }
}

storagepolicy(*size, *privacy, *availability){
        on(*size=="large"){ "archive"; }
}

storagepolicy(*size, *privacy, *availability){
        "demoResc";
}

INPUT *size="large", *privacy="low", *availability="high"
OUTPUT ruleExecOut
```

## Implement your own data archiving policy (30min)
- Automatically synchronise the *archive* collection to *bobZone*. Watch out! Only replicate your *archive* collection not your neighbours collection.
- Create metadata to track the data.

### Replication part
Have a look at the file *exampleRules/replicationPart.r* and fill in the missing parts.

```py
myReplicationPolicy{
    # create base path to your home collection and extend with what you want to replicate
    *source="/$rodsZoneClient/home/$userNameClient/<FILL_IN>";
    # by default we stay in the same iRODS zone and use a new collection called 'test'
    if(*destination == ""){ *destination = "/$rodsZoneClient/home/$userNameClient/test"}
    # some sanity checking
    writeLine("stdout", "Replicate *source");
    writeLine("stdout", "Destination *destination");

    replicate("*source", *destination, *syncStat)
    writeLine("stdout", "Irsync finished with: *syncStat");
}

replicate(*source, *dest, *status){
    # check whether it is a collection (-c) or a data object (-d)
    # *source_type catches return value of the function
    msiGetObjType(*source,*source_type);
    writeLine("stdout", "*source is of type *source_type");

    # Only proceed when source_type matches "collection"
    if(<FILL_IN>){
        msiCollRsync(*source, *destination,
            "null","IRODS_TO_IRODS",*status);
        writeLine("stdout", "Irsync status: *status");
    }
    else{
        # Create some useful message on the prompt
        writeLine("stdout", "<FILL_IN>");
        # Propagate the status variable so that it can be taken up by myReplicationPolicy
        *status = <FILL_IN>
    }
}

INPUT *coll="archive", *destination=""
OUTPUT ruleExecOut
```

### Attaching metadata
Open *exampleRules/metadataPart.r* and fill in the missing pieces.

```py
myMetadataPolicy{
    # Build absolute path for object or collection to label with metadata
    *path=<FILL_IN>
    writeLine("stdout", "Labeling *path");

    # Add metadata on TYPE, defined by system
    addMD("TYPE", "", *path)
    # Add user metadata
    writeLine("stdout", "*mdkey *mdval");
    addMD(*mdkey, *mdval, *path)
}

 # Function to attach metadata to any data collection or data object
 # Case 1: Metadata to extract from system --> TYPE
addMD(*key, *value, *path){
    on(<FILL_IN>){
        msiGetObjType(*path,*source_type);
        if(*source_type=="-d"){
            *MDValue="data object";
        }
        else{
            *MDValue="collection"
        }
        createAVU(*key, *MDValue, *path);
    }
}

 # Case 2: User defined metadata
addMD(*key, *value, *path){
    # Do not add metadata with empty value!
    # Test whether value is empty --> ""
    # Create AVU when value is given.
    <FILL_IN>
}

 # Low-level helper function
createAVU(*key, *value, *path){
    #Creates a key-value pair and connects it to a data object or collection
    msiAddKeyVal(*Keyval,*key, *value);
    writeKeyValPairs("stdout", *Keyval, " is : ");
    msiGetObjType(*path,*objType);
    msiSetKeyValuePairsToObj(*Keyval, *path, *objType);
}

INPUT *item="archive", *mdkey="ORIGINAL", *mdval="/aliceZone/home/di4r-user1/archive"
OUTPUT ruleExecOut
```

### Putting it all together: Replicate and link original and replica by appropriate metadata
Open *exampleRules/replication.r*. The file consists of the two parts and rules which we already created. Now link them in one workflow.

```py
replication{
    # create base path to your home collection
    *source=<FILL_IN>;
    # by default we stay in the same iRODS zone
    if(*destination == ""){ *destination = "/$rodsZoneClient/home/$userNameClient/test"}

    writeLine("stdout", "Replicate *source");
    writeLine("stdout", "Destination *destination");

    replicate("*source", *destination, *syncStat);
    writeLine("stdout", "Irsync finished with: *syncStat");
    writeLine("stdout", "");

    # Use addMD to link the original collection and the replicated collection
    # Create metadata for *collection
    writeLine("stdout", "Create metadata for input collection *source.")
    <FILL_IN>

    # Create metadata for *destination
    writeLine("stdout", "Create metadata for replica collection *destination.")
    <FILL_IN>
    writeLine("stdout", "");

    # Loop over all data objects in your archive collection in aliceZone
    writeLine("stdout", "Create metadata for all data objects in *source.")
    foreach(*row in SELECT COLL_NAME, DATA_NAME where COLL_NAME like "*source"){
        *coll = *row.COLL_NAME;
        *data = *row.DATA_NAME;
        *repl = <FILL_IN>; # build the paths to the original data file and the replica
        *orig = <FILL_IN>;

        linkOrigRepl(*path, *orig);
    }

    # Do the same for the sub collections
    writeLine("stdout", "Create metadata for all subcollections in *source.")
    foreach(*row in SELECT COLL_NAME where COLL_NAME like "*source/%"){
        *coll = *row.COLL_NAME;
        msiSplitPath(*coll, *parent, *child) #might be handy, have a look at the produced variables *parent and *child
        *repl = <FILL_IN>;

        linkOrigRepl(*coll, *repl);
    }
}
 # Given the original path and the replication path introduce the linking.
linkOrigRepl(*orig, *repl){
    # label orig with "REPLICA" *repl
    writeLine("stdout", "Metadata for *orig:");
    addMD("REPLICA", *repl, *orig);
    addMD("TYPE", "", *orig)
    writeLine("stdout", "");

    # label repl with "ORIGINAL" *orig
    writeLine("stdout", "Metadata for *repl:");
    addMD("ORIGINAL", *orig, *repl);
    addMD("TYPE", "", *repl)
    writeLine("stdout", "");
}

addMD(*key, *value, *path){
    on(*key=="TYPE"){
        msiGetObjType(*path,*source_type);
        if(*source_type=="-d"){
            *MDValue="data object";
        }
        else{
            *MDValue="collection"
        }
        createAVU(*key, *MDValue, *path);
    }
}

addMD(*key, *value, *path){
    # Do not add metadata with empty value!
    if(*value==""){
        writeLine("stdout", "No mdval given.");
    }
    else{
        createAVU(*key, *value, *path);
    }
}

createAVU(*key, *value, *path){
    msiAddKeyVal(*Keyval,*key, *value);
    writeKeyValPairs("stdout", *Keyval, " is : ");
    msiGetObjType(*path,*objType);
    msiSetKeyValuePairsToObj(*Keyval, *path, *objType);
}

replicate(*source, *dest, *status){
    # check whether it is a collection (-c) or a data object (-d)
    # *source_type catches return value of the function
    msiGetObjType(*source,*source_type);
    writeLine("stdout", "*source is of type *source_type");

    # Only proceed when source_type matches "collection"
    if(*source_type == "-c"){
        msiCollRsync(*source, *destination,
            "null","IRODS_TO_IRODS",*status);
    }
    else{
       writeLine("stdout", "Expected Collection, got data object.");
       *status = "FAIL - No data collection."
    }
}
```

#### Solution
```
replication{
    # create base path to your home collection
    *source="/$rodsZoneClient/home/$userNameClient/*collection";
    # by default we stay in the same iRODS zone
    if(*destination == ""){ *destination = "/$rodsZoneClient/home/$userNameClient/test"}

    writeLine("stdout", "Replicate *source");
    writeLine("stdout", "Destination *destination");

    replicate("*source", *destination, *syncStat);
    writeLine("stdout", "Irsync finished with: *syncStat");
    writeLine("stdout", "");

    # Create metadata for *collection
    writeLine("stdout", "Create metadata for input collection *source.")
    addMD("TYPE", "", *source);
    addMD("REPLICA", *destination, *source);
    # Create metadata for *destination
    writeLine("stdout", "Create metadata for replica collection *destination.")
    addMD("TYPE", "", *destination);
    addMD("ORIGINAL", *source, *destination);
    writeLine("stdout", "");

    # Loop over all data objects in your archive collection in aliceZone
    writeLine("stdout", "Create metadata for all data objects in *source.")
    foreach(*row in SELECT COLL_NAME, DATA_NAME where COLL_NAME like "*source"){
        *coll = *row.COLL_NAME;
        *data = *row.DATA_NAME;
        *repl = *destination++"/"++*data;
        *path = *coll++"/"++*data;

        linkOrigRepl(*path, *repl);
    }

    # Do the same for the sub collections
    writeLine("stdout", "Create metadata for all subcollections in *source.")
    foreach(*row in SELECT COLL_NAME where COLL_NAME like "*source/%"){
        *coll = *row.COLL_NAME;
        msiSplitPath(*coll, *parent, *child)
        *repl = *destination++"/"++*child;

        linkOrigRepl(*coll, *repl);
    }
}

...

INPUT *source="archive", *destination="/bobZone/home/alice#aliceZone"
OUTPUT ruleExecOut
```

