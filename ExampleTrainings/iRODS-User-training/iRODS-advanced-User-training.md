# iRODS advanced User Training

- Recap on icommands
- iRODS federations
- iRODS rule language and write your own backup data policy


## Recap icommands (15 min)

|                      Command                       |                                Meaning                                 |
|----------------------------------------------------|------------------------------------------------------------------------|
| iinit                                              | Login                                                                  |
| ienv                                               | iRODS environment                                                      |
| iuserinfo                                          | User attributes                                                        |
| **ihelp**                                          | List of all commands                                                   |
| **\<command\> -h**                                 | Help                                                                   |
| **Up- and down load**                              |                                                                        |
| iput [-K -r -f -R \<resc\>]                        | Upload data, create checksum, recursively, overwrite, specify resource |
| iget [-K -r -f]                                    | Check checksum, recursively, overwrite                                 |
| **Data organisation**                              |                                                                        |
| ils [-L -A -l]                                     | List collection [long format, access control list, less long format]   |
| imkdir                                             | Create collection                                                      |
| icd                                                | Change current working collection                                      |
| **Metadata**                                       |                                                                        |
| imeta add [-d -C] Name AttName AttValue [AttUnits] | Create metadata [file, collection]                                     |
| imeta ls [-d -C]                                   | List metadata [file, collection]                                       |
| iquest                                             | Find data by query on metadata                                         |
| iquest attrs                                       | List of attributes to query                                            |

**Some predefined attributes for iquest:**

USER\_ID, USER\_NAME, RESC\_ID, RESC\_NAME, RESC\_TYPE\_NAME, RESC\_CHILDREN,
RESC\_PARENT, DATA\_NAME, DATA\_REPL\_NUM, DATA\_SIZE, DATA\_RESC\_NAME,
DATA\_PATH, DATA\_OWNER\_NAME, DATA\_CHECKSUM, COLL\_ID, COLL\_NAME,
COLL\_PARENT\_NAME, COLL\_OWNER,\_NAME META\_DATA\_ATTR\_NAME,
META\_DATA\_ATTR\_VALUE, META\_DATA\_ATTR\_UNITS, META\_DATA\_ATTR\_ID,
META\_COLL\_ATTR\_NAME, META\_COLL\_ATTR\_VALUE, META\_COLL\_ATTR\_UNITS,
META\_COLL\_ATTR\_ID, META\_COLL\_CREATE\_TIME, META\_COLL\_MODIFY\_TIME,
META\_NAMESPACE\_COLL, META\_RESC\_ATTR\_NAME, META\_RESC\_ATTR\_VALUE,
META\_RESC\_ATTR\_UNITS

**Example query**:

```
iquest "select COLL_NAME, DATA_NAME where \
META_DATA_ATTR_NAME like 'author' and META_DATA_ATTR_VALUE like 'Alice'"
```

Remember: in *iquest*, the percent sign (%) is the wildcard character that
matches zero or more characters.

### Login

You login to iRODS with the command

```
iinit
```

You will be asked for the iRODS server you would like to connect to the port
(standard 1247), the zone name of the iRODS server, your iRODS user name and
password.

### Basic commands

First we will have a look at some very basic commands to move through the
logical namespace in iRODS. The basic commands in iRODS are very similar to
bash/shell commands. You can browse through your collections with:

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

The flag *-K* triggers the calculation and verification of a checksum, in this case an MD5 checksum.
Now upload a collection to iRODS:

```
iput -r -K aliceInWonderland lewiscarroll/book-aliceInWonderland
```

### Logical and physical namespaces in iRODS

The *ils* command gives you an option to extract the physical location of a
file with the *-L* flag:

```
ils -L -r lewiscarroll/book-aliceInWonderland
```

You will see some output similar to:

```
alice               0 demoResc           29 2016-08-23.09:04
& aliceInWonderland-EN.txt.utf-8
    9dcbb372c049bdd4b035c1ccb3798e69    generic
    /irodsVault/home/alice/lewiscarroll/book-aliceInWonderland/aliceInWonderland-EN.txt.utf-8
```

We will use this command quite often today to see what happens with files upon
replication. With this command you see where the file is stored on the iRODS
server.

- *alice* is the data owner.
- *0* is the index of this replica. The number only refers to replicas in one
  iRODS zone and can be used to automatically trim the number of replicas or
  create new ones in case one got lost.
- *demoResc* is the resource on which the data is stored. Resources can refer
  to certain paths on the iRODS server or other storage servers and clusters.

- The next entry is the time of the upload and the file name.
- The following entry is the checksum, in our case it is an MD5 checksum.
- The last entry is the physical path of the data, in our case the data lies on
  the iRODS server.

### Exercise (5 min)

Data can be downloaded from iRODS to your local machine with the command
*iget*. Explore the command *iget* to **store the data in you home directory**.
Do **not overwrite** your original data and do **verify checksums**!


## iRODS federations (10 min)

iRODS federations are connections between different iRODS servers or - in iRODS
terms - *zones*. iRODS federations are setup by the system administrators. They
also exchange users which allows you as a user to read and write data at a
different iRODS zones.

In our example your useraccounts are known and authenticated at *aliceZone*.
Each of your accounts has a counterpart at the remote zone *bobZone*.

Let us have a look at how we can access our **remote** home directory.

```
ils /bobZone/home/di4r-user1#aliceZone
/bobZone/home/di4r-user1#aliceZone:
```

Note that when you are accessing your remote home you have to state at which
iRODS zone you are authenticated. This is indicated with the *#aliceZone*.

We can put data directly from your UI Linux account in the remote home:

```
iput -K aliceInWonderland-DE.txt.utf-8 /bobZone/home/di4r-user1#aliceZone
```

```
ils /bobZone/home/di4r-user1#aliceZone -L
/bobZone/home/di4r-user1#aliceZone:
  di4r-user1        0 demoResc       187870 2017-03-27.10:20
  & aliceInWonderland-DE.txt.utf-8
    7bdfc92a31784e0ca738704be4f9d088    generic
    /irodsVault/home/di4r-user1#aliceZone/aliceInWonderland-DE.txt.utf-8
```

#### Small exercise (10min)

- Try to use *imv* to move *aliceInWonderland-DE.txt.utf-8* from *aliceZone* to
  *bobZone*. Can you use *icp*? What could be the reasoning for the different
  behaviour?
- Download the German version from *bobZone* to the 'local' Unix filesystem (on
  the UI machine). Store it under a different file name. Which commands can you
  use?

The command *imv* edits the corresponding entry in the iCAT metadata catalogue
at *aliceZone* and moves the data physically to a new location in the *Vault*.

```
ils -L aliceInWonderland-DE.txt.utf-8
imv aliceInWonderland-DE.txt.utf-8 aliceGerman.txt
ils -L aliceGerman.txt
```

*imv* would mean that the metadata entry is at *aliceZone*, while the data is
physically stored at *bobZone*. With *icp* you create a new data object with a
metadata entry at its iRODS zone and storage site.

### Replicating and synchronising data (15min)

As with *gridFTP* and *rsync* iRODS offers a command to synchronise data
between either your 'local' Unix filesystem or between different iRODS zones.
In contrast to pure iRODS replication with *irepl* this will create new data
objects and collections at the remote zone.

In the following we will use the remote iRODS zone as a backup server. The
iRODS collection *archive* will serve as source collection for the backup.

Upload one of the files in *aliceInWonderland* to your home collection on
**aliceZone**. Do not use the *-K* flag.

```
iput aliceInWonderland/aliceInWonderland-EN.txt.utf-8
```

We can replicate the file to *bobZone*

```
irsync i:/aliceZone/home/di4r-user1/aliceInWonderland-EN.txt.utf-8 \
i:/bobZone/home/di4r-user1#aliceZone/aliceInWonderland-EN.txt.utf-8
```

Check

```
ils -L /bobZone/home/di4r-user1#aliceZone/aliceInWonderland-EN.txt.utf-8
```

You see that at the remote site there is a checksum calculated. *irsync*
calculates the checksums and uses them to determine whether the file needs to
be transferred. After some delay you will also see with *ils -L* on
**aliceZone** that iROD calculated and stored a checksum for the file. File
transfers with *irsync* are faster when first calculating the checksum and
then transferring them.

#### Small Exercise

Which commands can you use to download the data from the remote iRODS zone to
your 'local' Unix filesystem?

### Exercise (15min)

Verify that *irsync* really just updates data when necessary.

1. Create a collection on *aliceZone*, e.g. *archive*
2. Add some files to this collection, e.g. the German version of Alice in
   Wonderland (use *icp* or *imv*).
3. Check what needs to be synchronised with *irsync -l* flag. What does this
   flag do?
4. Synchronise the whole collection with *bobZone* (not only the file). Which
   flag do you have to use?
5. Check again if there is something to be synchronised.
6. Add another file to *archive* on *aliceZone*, e.g. the Italian version of
   Alice in Wonderland.
7. Check the synchronisation status. (It can take some time until the iRODS
   system marks the new files as 'synchronised')

#### Solution

1. Synchronising

 ```
 imkdir archive
 icp -K aliceInWonderland-DE.txt.utf-8 archive
 irsync -r i:archive i:/bobZone/home/di4r-user1#aliceZone/archive
 irsync -r -l i:archive i:/bobZone/home/di4r-user1#aliceZone/archive
 ```
2. Add new files to *archive*

 ```
 iput aliceInWonderland/aliceInWonderland-IT.txt.utf-8 archive/
 ```
3. Check sync-status

 ```
 irsync -r -l i:archive i:/bobZone/home/di4r-user1#aliceZone/archive
 /aliceZone/home/di4r-user1/archive/aliceInWonderland-IT.txt.utf-8   175251   N
 ```
4. Synchronising and checking sync-status

 ```
 irsync -r i:archive i:/bobZone/home/di4r-user1#aliceZone/archive
 irsync -r -l i:archive i:/bobZone/home/di4r-user1#aliceZone/archive
 ```

If you did not calculate the checksums for the source files, the sync-status
needs some time to be updated.

### Metadata for remote data (5min)

We created another copy of the *archive* collection at *bobZone* but we lost
the link to the data at *aliceZone*.  We will now have a look at how we can use
the iCAT metadata catalogues at *bobZone* and at *aliceZone* to link the data.

Recall, we can create metadata for iRODS data objects and collections on our
home iRODS zone like this:

```
imeta add -C archive "Original" "/aliceZone/home/di4r-user1/archive"
imeta add -d archive/aliceInWonderland-DE.txt.utf-8 \
"Original" "/aliceZone/home/di4r-user1/archive/aliceInWonderland-DE.txt.utf-8"
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
imeta add -C /bobZone/home/di4r-user1#aliceZone/archive \
"Original" "/aliceZone/home/di4r-user1/archive"
```

#### Small exercise (5min)

1. Label the files in */bobZone/home/di4r-user1#aliceZone/archive* with
   information on its original source.
2. Introduce another metadata field in the original data to link to the
   replicas. Use the key "Replica".

#### Solution

```
imeta add -C archive "Replica" "/bobZone/home/di4r-user1#aliceZone/archive"
imeta add -d archive/aliceInWonderland-DE.txt.utf-8 \
"Replica" "/bobZone/home/di4r-user1#aliceZone/archive/aliceInWonderland-DE.txt.utf-8"
```


### Retrieving data by metadata (10min)

We can retrieve our freshly labeled data at *aliceZone*

```
iquest "select COLL_NAME, DATA_NAME where META_DATA_ATTR_NAME like 'Original'"
COLL_NAME = /aliceZone/home/di4r-user1/archive
DATA_NAME = aliceInWonderland-EN.txt.utf-8
```

To query the metadata in the iCAT catalogue for *bobZone* we need to specify
this using the *-z* flag of *iquest*.

```
iquest "select COLL_NAME, DATA_NAME where \
META_DATA_ATTR_NAME like 'Original' and COLL_NAME like '%bobZone%'"

CAT_NO_ROWS_FOUND: Nothing was found matching your query


iquest -z bobZone "select COLL_NAME, DATA_NAME where \
META_DATA_ATTR_NAME like 'Original' and COLL_NAME like '%bobZone%'"
Zone is bobZone
COLL_NAME = /bobZone/home/di4r-user1#aliceZone/archive
DATA_NAME = aliceInWonderland-EN.txt.utf-8
```

There is no standard way to query multiple iCAT catalogues with a single
*iquest* command.

#### Small exercise (10 min)

Assume your "Original" *archive* collection on *aliceZone* is corrupted. How
would you find out where you can get a copy of that data and how would you
restore the data?

#### Solution

```
iquest \
"select META_COLL_ATTR_VALUE where META_COLL_ATTR_NAME like 'Replica' and COLL_NAME like '%archive%'"
irsync -r -l i:\<answer from iquest\> i:/aliceZone/home/di4r-user1/archive
irsync -r i:\<answer from iquest\> i:/aliceZone/home/di4r-user1/archive
```

## iRODS rules (30min)
In the previous parts we did a lot of work manually:

- replicating data to a different zone
- labeling data to keep track of originals and replicas

iRODS offers the possibility to automate data management processes by creating scripts written in the iRODS rule language. We will first inspect the iRODS rule language and then automatise the steps of the previous section and finally schedule the backup process in regular time intervals.

Open *exampleRules/helloworld.r*

```c
HelloWorld{
	writeLine("stdout", "Hello *name!");
}

INPUT *name="YourName"
OUTPUT ruleExecOut, *name
```

and execute the rule with

```
irule -F exampleRules/helloworld.r
```

Rules are structured as follows:

- Name of the rule, this name does not have to correspond with the filename
- Curly braces to indicate code blocks
- User defined rules always need an *INPUT* and *OUTPUT* at the end of the file, each in a separate line
	- INPUT: Defines some variables from the commandline
	- OUTPUT: always needs the *ruleExecOut* and can contain variables that will be printed to the prompt

### iRODS microservices, rules, policy-enforcement points and the rulebase
**Slides**
- Microservices
- Rule engine
- The default rulebase *core.re*
- Order matters
- Workflow for developing rules

### Passing arguments, variables and output
The rule has an input variable which we did not set in the previous call. The default value for the variable is "YourName".
To customise the function, we could alter the code, or we could pass on the right value for the variable.

```c
HelloWorld{
	writeLine("stdout", "Hello *name!");
}

INPUT *name="YourName"
OUTPUT ruleExecOut, *name
```

We can overwrite input parameters by calling the function like this:

```
irule -F exampleRules/helloworld.r "*name='Alice'"
```

### Exercise: Passing variables, data types (10min)

```c
variables{
	writeLine("stdout", "var1 is *var1!");
	writeLine("stdout", "var2 is *var2!");
}

INPUT *var1=1, *var2="string"
OUTPUT ruleExecOut, *name
```

- Alter the type of the input variables: numbers and simple calculations, booleans (true, false), strings
- How do you only change one of the two variables?

```
irule -F exampleRules/variables.r '*var1="Hello"'
```

```
irule -F exampleRules/variables.r "*var1='456'" "*var2='true'"
```

```
irule -F exampleRules/variables.r '*var1=3+4/7.'
```

```
irule -F exampleRules/variables.r "*var1='Hello'" "*var2=Hello"
```

### Global/System variables

iRODS knows predefined global variables that are set by the system and can come in handy. Those variables are addressed by "$" just like in shell scripting. 
With them you can e.g. create the home collection of the active user:

```c
*home="/$rodsZoneClient/home/$userNameClient"
```

### Looping over data

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
```
irule -F exampleRules/recursivelist.r
```
The '%' works as wild card, variables are denoted by '*'.

We see that this kind of for-loops use statements similar to the ones in the *iquest* command to retrieve data and collections.

### Exercise (15min)
Write a rule that finds all data objects and all collections that carry the same metadata key.
E.g. there is a collection labeled with the attribute 'RIDDLE' and there are some files carrying the same attribute. Make the attribute a variable.

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

input *var='RIDDLE'
output ruleExecOut
```

### Solution

```c
queryall{
        foreach(*row in SELECT COLL_NAME, META_COLL_ATTR_VALUE where 
        	META_COLL_ATTR_NAME like '*var'){
        *coll = *row.COLL_NAME;
        *value = *row.META_COLL_ATTR_VALUE;
        writeLine("stdout", "*coll *value");
        }
        foreach(*row in SELECT COLL_NAME, DATA_NAME, META_DATA_ATTR_VALUE where 
        	META_DATA_ATTR_NAME like '*var'){
        *coll = *row.COLL_NAME;
        *data = *row.DATA_NAME;
        *value = *row.META_DATA_ATTR_VALUE;
        writeLine("stdout", "*coll/*data *value");
        }

}

input *var='RIDDLE'
output ruleExecOut
```

### If-statements and on-statements
Open *exampleRules/conditionalhello.r*.

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
The same rule above looks like this with on-statements (*exampleRules/conditionalhelloon.r*):

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
- Switch the two *hello* rules in the rule above. What happens and why?

- Write a rule with different cases (decision between data policies), set the variable "iresource" accordingly:
	- If the data size is large, "iresource" should be "archive"
	- If the data should be highly available, "iresouce" should be "replResc"
	- If the data is classified as sensitive data, "iresource" should always be "storage3"
	- In all other cases the data should go to the "demoResc"
	
- Implement the policies using *if* or *on*.
- Which of the two is more advantageous if you think of what you need to alter when one of the cases (policies) changes?
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

### Return values of rules
We saw in the previous exercise that functions automatically return the last value that is set (not assigned to another variable!). But how can you pass on variables or several variables? An alternative to the solution above is he following:

```c
policydecision{
        storagepolicy(*size, *privacy, *availability, *resourceName);
        writeLine("stdout", "*resourceName");
}

storagepolicy(*size, *privacy, *availability, *resource){
        on(*privacy=="high"){ *resource = "storage3"; }
}

storagepolicy(*size, *privacy, *availability, *resource){
        on(*availability=="high"){ *resource = "replResc"; }
}

storagepolicy(*size, *privacy, *availability, *resource){
        on(*size=="large"){ *resource = "archive"; }
}

storagepolicy(*size, *privacy, *availability, *resource){
        *resource = "demoResc";
}

INPUT *size="large", *privacy="low", *availability="high"
OUTPUT ruleExecOut
```

Here we create a dummy variable *resourceName* that is empty when the *storagepolicy* is called and which is assigned with *resource* after the subpolicies are executed.

**Show both solutions next to each other.**

### Implement your own data archiving policy (60min)

The data archiving rule should consist of two rules (policies).

1. Automatically synchronise the *archive* collection to *bobZone*. Watch out! Only replicate your *archive* collection not your neighbours collection.
2. Create metadata to track the data.

We will give examples for replicating collections.

#### Exercise: The replication part
Write a rule that will replicate iRODS collections (template in 
*exampleRules/replicationPart.r*).

```c
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

#### Solution - replication part

```c
myReplicationPolicy{
    # create base path to your home collection
    *source="/$rodsZoneClient/home/$userNameClient/*coll";
    # by default we stay in the same iRODS zone
    if(*destination == ""){ *destination = "/$rodsZoneClient/home/$userNameClient/test"}

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
    if(*source_type == "-c"){
        msiCollRsync(*source, *destination,
            "null","IRODS_TO_IRODS",*status);
        writeLine("stdout", "Irsync status: *status");
    }
    else{
       writeLine("stdout", "Expected Collection, got data object.");
       *status = "FAIL - No data collection."
    }
}

INPUT *coll="archive", *destination=""
OUTPUT ruleExecOut
```

Example output:
```
irule -F exampleRules/replicationPart_solution.r
Replicate /aliceZone/home/di4r-user1/archive
Destination /aliceZone/home/di4r-user1/test
/aliceZone/home/di4r-user1/archive is of type -c
Irsync status: 0
Irsync finished with: 0

irule -F exampleRules/replicationPart_solution.r \
	"*coll='lewiscarroll/book-aliceInWonderland/aliceInWonderland-EN.txt.utf-8'"
Expected Collection, got data object.
Irsync finished with: FAIL - No data collection.
```

#### Exercise: The metadata part
Write a rule that attaches metadata to collections and data objects (template in *exampleRules/metadataPart.r*).
The metadata should contain a key-value pair determining whether the data is a collection or a data object (extract that information automatically from iRODS), another metadata entry should be determined by a key value pair given as input of the rule.

```c
myMetadataPolicy{

    # Build absolute path for obaject or collection to label with metadata
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

#### Solution - metadata part
```c
myMetadataPolicy{

    # Build absolute path for obaject or collection to label with metadata
    *path="/$rodsZoneClient/home/$userNameClient/*item"
    writeLine("stdout", "Labeling *path");

    # Add metadata on TYPE, defined by system
    addMD("TYPE", "", *path)
    # Add user metadata
    writeLine("stdout", "*mdkey *mdval");
    addMD(*mdkey, *mdval, *path)
}

# Function to attach metadata to any data collection or data object
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
Example output:

```
irule -F exampleRules/metadataPart_solution.r
Labeling /aliceZone/home/di4r-user1/archive
TYPE is : collection
ORIGINAL is : /aliceZone/home/di4r-user1/archive

irule -F exampleRules/metadataPart_solution.r "*item='archive/aliceInWonderland-DE.txt.utf-8'"
Labeling /aliceZone/home/di4r-user1/archive/aliceInWonderland-DE.txt.utf-8
TYPE is : data object
ORIGINAL is : /aliceZone/home/di4r-user1/archive

irule -F exampleRules/metadataPart_solution.r "*item='archive/aliceInWonderland-DE.txt.utf-8'" "*mdval=''"
Labeling /aliceZone/home/di4r-user1/archive/aliceInWonderland-DE.txt.utf-8
TYPE is : data object
ORIGINAL
No mdval given.
```
**Watch out:** Do not use special characters in metadata entries. I.e. no signs that have a special meaning in iRODS: '-', '$', '#', ...


#### Exercise: Putting it all together
Now that we have the single parts, write a rule, that replicates a collection to *bobZone* and labels the collection and all its contents:

- The original data on *aliceZone* with the links to the replicated data on *bobZone*
 
 ```
attribute: REPLICA
value: /bobZone/home/di4r-user1#aliceZone/archive
 ```
- The replicated data on *bobZone* with the link to the original data on *aliceZone*

 ```
 attribute: ORIGINAL
 value: /aliceZone/home/di4r-user1/archive
 ```
 
- **Start fresh**: Make sure you have some data and some subcollection in an iRODS collection called *archive*
 
 ```
 ils archive

 /aliceZone/home/di4r-user1/archive:
 aliceInWonderland-DE.txt.utf-8
 C- /aliceZone/home/di4r-user1/archive/aliceInWonderland
  
 imeta ls -C archive

 AVUs defined for collection archive:
 None
 ```
 
```c
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
    foreach(*row in SELECT COLL_NAME where COLL_NAME like "%archive/%"){
        *coll = *row.COLL_NAME;
        #might be handy, have a look at the produced variables *parent and *child
        msiSplitPath(*coll, *parent, *child) 
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

INPUT *collection="archive", *destination="/bobZone/home/di4r-user1#aliceZone/BACKUP"
OUTPUT ruleExecOut
 ```

#### Solution
```c
replication{
    # create base path to your home collection
    *source="/$rodsZoneClient/home/$userNameClient/*collection";
    # by default we stay in the same iRODS zone
    if(*destination == ""){ *destination = "/$rodsZoneClient/home/$userNameClient/test"}

    writeLine("stdout", "Replicate *source");
    writeLine("stdout", "Destination *destination");

    replicate("*source", *destination, *syncStat)
    writeLine("stdout", "Irsync finished with: *syncStat");
    writeLine("stdout", "")

    # Create metadata for *collection
    writeLine("stdout", "Create metadata for input collection *source.")
    addMD("TYPE", "", *source);
    addMD("REPLICA", *destination, *source);
    # Create metadata for *destination
    writeLine("stdout", "Create metadata for replica collection *destination.")
    addMD("TYPE", "", *destination);
    addMD("ORIGINAL", *source, *destination);
    writeLine("stdout", "")

    # Loop over all data objects in your archive collection in aliceZone
    # Set the field REPLICA for the data objects in archive
    # Set the field ORIGINAL for the data objects in the replicas
    writeLine("stdout", "Create metadata for all data objects in *source.")
    foreach(*row in SELECT COLL_NAME, DATA_NAME where COLL_NAME like "*source"){
        *coll = *row.COLL_NAME;
        *data = *row.DATA_NAME;
        *repl = *destination++"/"++*data;
        *path = *coll++"/"++*data;

        linkOrigRepl(*path, *repl)
    }

    # Do the same for the sub collections
    writeLine("stdout", "Create metadata for all subcollections in *source.")
    foreach(*row in SELECT COLL_NAME where COLL_NAME like "%archive/%"){
        *coll = *row.COLL_NAME;
        msiSplitPath(*coll, *parent, *child)
        *repl = *destination++"/"++*child;

        linkOrigRepl(*coll, *repl)
    }
}

linkOrigRepl(*orig, *repl){
    writeLine("stdout", "Metadata for *orig:");
    addMD("REPLICA", *repl, *orig);
    addMD("TYPE", "", *orig)
    writeLine("stdout", "");

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

INPUT *collection="archive", *destination=""
OUTPUT ruleExecOut

```

Example output:

```
di4r-user1@ui:~$ irule -F exampleRules/replication.r
Replicate /aliceZone/home/di4r-user1/archive
Destination /bobZone/home/di4r-user1#aliceZone/BACKUP
/aliceZone/home/di4r-user1/archive is of type -c
Irsync finished with: 0

Create metadata for input collection /aliceZone/home/di4r-user1/archive.
TYPE is : collection
REPLICA is : /bobZone/home/di4r-user1#aliceZone/BACKUP
Create metadata for replica collection /bobZone/home/di4r-user1#aliceZone/BACKUP.
TYPE is : collection
ORIGINAL is : /aliceZone/home/di4r-user1/archive

Create metadata for all data objects in /aliceZone/home/di4r-user1/archive.
Metadata for /aliceZone/home/di4r-user1/archive/aliceInWonderland-DE.txt.utf-8:
REPLICA is : /bobZone/home/di4r-user1#aliceZone/BACKUP/aliceInWonderland-DE.txt.utf-8
TYPE is : data object

Metadata for /bobZone/home/di4r-user1#aliceZone/BACKUP/aliceInWonderland-DE.txt.utf-8:
ORIGINAL is : /aliceZone/home/di4r-user1/archive/aliceInWonderland-DE.txt.utf-8
TYPE is : data object

Create metadata for all subcollections in /aliceZone/home/di4r-user1/archive.
Metadata for /aliceZone/home/di4r-user1/archive/aliceInWonderland:
REPLICA is : /bobZone/home/di4r-user1#aliceZone/BACKUP/aliceInWonderland
TYPE is : collection

Metadata for /bobZone/home/di4r-user1#aliceZone/BACKUP/aliceInWonderland:
ORIGINAL is : /aliceZone/home/di4r-user1/archive/aliceInWonderland
TYPE is : collection
```

## Last Challenge
**The big cleanup:** Write a policy, that removes all data from your iRODS home collection and from your remote home collection.

Hints:

- Loop over all data objects AND collections in aliceZone
- Loop over all data objects AND collections in bobZone
- Microservices to delete data objects and collections:
	- `msiDataObjUnlink(*data,*status)`
	- `msiRmColl(*collection, "", *status)`

### Solution
```c
cleanup{
    *home="/$rodsZoneClient/home/$userNameClient";
    *remote="/bobZone/home/$userNameClient#aliceZone";
    writeLine("stdout", *home);
    writeLine("stdout", $userNameClient);
    #data
    writeLine("stdout", "Cleanup *remote")
    foreach(*row in SELECT COLL_NAME, DATA_NAME where COLL_NAME like "*home%"){
        *path = *row.COLL_NAME++'/'++*row.DATA_NAME;
        writeLine("stdout", "Remove *path");
        msiGetObjType(*path,*objType);
        remove(*objType, *path);
    }
    foreach(*row in SELECT COLL_NAME where COLL_NAME like "*home/%"){
        *path = *row.COLL_NAME;
        writeLine("stdout", "Remove *path");
        msiGetObjType(*path,*objType);
        remove(*objType, *path);
    }

    writeLine("stdout", "Cleanup *home")
    foreach(*row in SELECT COLL_NAME, DATA_NAME where COLL_NAME like "*remote%"){
        *path = *row.COLL_NAME++'/'++*row.DATA_NAME;
        writeLine("stdout", "Remove *path");
        msiGetObjType(*path,*objType);
        remove(*objType, *path);
    }
    foreach(*row in SELECT COLL_NAME where COLL_NAME like "*remote/%"){
        *path = *row.COLL_NAME;
        writeLine("stdout", "Remove *path");
        msiGetObjType(*path,*objType);
        remove(*objType, *path);
    }
}

remove(*objType, *data){
    on($userNameClient == "rods") { writeLine("stdout", "You are admin. I refuse to delete your data."); }
}

remove(*objType, *data){
    on($userNameClient == "alice") { writeLine("stdout", "You are admin. I refuse to delete your data."); }
}

remove(*objType, *data){
    on($userNameClient == "bob") { writeLine("stdout", "You are admin. I refuse to delete your data."); }
}

remove(*objType, *data){
    on(*objType == '-d') {msiDataObjUnlink("*data",*status); writeLine("stdout", "Done.");}
}

remove(*objType, *data){
    on(*objType == '-c') {msiRmColl(*data, "", *status); }
}
INPUT null
OUTPUT ruleExecOut

```

