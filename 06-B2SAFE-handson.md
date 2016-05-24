# EUDAT B2SAFE hands-on
This hands-on will illustrate how B2SAFE rules can be employed to manage data across iRODS zones by policies.
The tutorial makes use of the icommands. If you did not so then please first follow the tutorial on [using iRODS](https://github.com/chStaiger/B2SAFE-B2STAGE-Training/blob/master/01-iRODS-handson-user.md).

## B2SAFE data transfer workflow (Using B2SAFE)
### To follow this tutorial it is advised to first follow the tutorial on using iRODS.

### Outline
The tutorial will guide you through Step2 in the figure below.
As B2SAFE admin you will copy data from a user, which he/she ingested into the iRODS instance, to another location in iRODS. You will register the data and by that buil the so-called repository of records and replicate the collection to another iRODS server using the B2SAFE rules.s B2SAFE admin you will copy data from a user, which he/she ingested into the iRODS instance, to another location in iRODS. You will register the data and by that, build the so-called repository of records and replicate the collection to another iRODS server using the B2SAFE rules.

<img align="center" src="img/workflow.png" width="500px">

### Prerequisites
- Installation of the icommands
- As iRODS user ingest data into iRODS and give your B2SAFE admin access to the collection. These steps are explained in the iRODS-using tutorial.

### iRODS rules
iRODS provides a way to execute data management procedures automatically and on regular bases or upon a certain action. To this end these procedures are defined in so-called iRODS rules.

A simple rule is:

```sh
HelloWorld {
  writeLine("stdout", "Hello, world!");
}
INPUT null
OUTPUT ruleExecOut
```

You can save this rule as hello.r and call it via the icommands:
```sh
irule -F hello.r
```
The option *-F* indicates that the next argument is a file.
iRODS provides some standard rules which you can find here
```sh
/etc/irods/core.re
```
In 
```sh
B2SAFE-core/rulebase/
```
You will find the B2SAFE rules, which you can use and combine to enable your data management workflow.

### Example: Using B2SAFE to register a file
You can use the B2SAFE rule *EUDATCreatePID* to register a single file. The rule is located in *B2SAFE-core/rulebase/pid-service.re*.

First let's ingest a data file into iRODS.
```sh
iput -f put1.txt
```
Now let's write a rule which calls *EUDATCreatePID* and registers our *put1.txt*.
```sh
registerFile {
        EUDATCreatePID(*parent_pid, *path, *ror, bool("false"), *newPID);
}
INPUT *path = "/aliceZone/home/alice/put1.txt", *ror = "", *parent_pid =""
OUTPUT *newPID, ruleExecOut
```
And save this file as testRules/registerFile.r
The rule takes our *put1.txt* as input file. We also communicate that there does not exist a repository or resources yet (\*ror). If the file we would like to register is a replica of another file, we can give the PID of the so-called parent with the parameter \*parent_pid to introduce the correct linking of the PID entries (see also the PID tutorial).
In *OUTPUT* we define which variables should be prompted on the command line, in this case we would like to receive the newly created PID.

Execute the rule:
```sh
irule -F testRules/registerFile.r
```
The answer will be the PID, e.g.:
```sh
*newPID = 846/b2f56c02-d987-11e5-8ef9-04040a64000c
```
This PID has been created at the SURFsara epic test server and can be resolved here: http://epic3.storage.surfsara.nl:8001
Enter the full PID string and tick the box *do not redirect to URLs*. This will show you the metadata stored with the PID. *URL* contains the iRODS path where to find the file. You will find that the B2SAFE rule also automatically calculated and stored a sha256 checksum. The field *100/LOC* also contains the location of our file. This field will become interesting when we create chains of replicas of files.

### B2SAFE Replication workflow


1. Copy the user's data to location und B2SAFE administrator home collection

        icp -r /aliceZone/home/<irodsuser>/DataCollection /aliceZone/home/b2safe/

2. Register all files in the collection using *EUDATPidsForColl*. Save the following file as testRules/eudatPidsColl.r and replace the user and collection name with your respective user and collection name.
        
        eudatPidsColl{
            # Create PIDs for all collections and objects in the collection recursively
            # ROR is assumed to be "None"
            EUDATPidsForColl(*coll_path);
        }
        INPUT *coll_path='/aliceZone/home/<b2safe>/<collection>'
        OUTPUT ruleExecOut
We see that here there is no output of the newly generated PIDs. However, we can retrieve this information by querying the iCAT catalogue.

        imeta ls -d DataCollection/put1.txt
This will return:

        attribute: eudat_dpm_checksum_date:demoResc
        value: 01455887784
        units:
        ----
        attribute: PID
        value: 846/6e67a674-d98a-11e5-b634-04040a64000c
        units:
**Exercise**: Write a script or an iRODS rule to retrieve all PIDs of a data collection.

3. Replicate the data collection from aliceZone to bobZone. The B2SAFE admin also has access to bobZone via an iRODS federation. We will now transfer the data collection to this zone.
Merely transferring the data could also be done by the icommand *irepl*. However, we would like to 1) calculate checksums, create PIDs and link the replicas' PIDs with their parent counterparts. This is all already implemented by B2SAFE rules.
Create the file testRules/Replication.r with the following content:
        
        Replication {
            *registered=bool("true");
            *recursive=bool("true");
            *status = EUDATReplication(*source, *destination, *registered, *recursive,  *response);
            if (*status) {writeLine("stdout", "Success!");}
            else {writeLine("stdout", "Failed: *response");}
        }
        INPUT *source="/aliceZone/home/<b2safe>/<collection>",*destination="/bobZone/home/<b2safe>#aliceZone/<collection>"
        OUTPUT ruleExecOut
Now let's have a closer look at the PID entries of the parent data on aliceZone. The resolver will show you some information like that:

[//] # "Perhaps a private ip address instead"
Index |  Type |   Timestamp |  Data
------|--------|--------------|--------
1 |  URL| 2016-02-22 17:33:49Z |   irods://145.100.58.12:1247/aliceZone/home/alice/DataCollection/put1.txt
2 |  10320/LOC |  2016-02-22 17:46:04Z |   \<locations\>\<location href="irods://145.100.58.12:1247/aliceZone/home/alice/DataCollection/put1.txt" id="0"/\>\<location href="http://hdl.handle.net/841/244bb240-d98c-11e5-aa5b-04040a640018" id="1"/\>\</locations>
3 |  CHECKSUM  |  2016-02-22 17:33:49Z |   d6eb32081c822ed572b70567826d9d9d

The *100/LOC* of the parent file has been extended with the PID of it's replica.

Let's have a look at the content of the replica's PID

[//] # "Perhaps a private ip address instead"
Index |  Type |   Timestamp |  Data
------|--------|--------------|--------
1 |  URL | 2016-02-22 17:46:04Z  |  irods://145.100.58.24:1247/bobZone/home/alice#aliceZone/DataCollection/put1.txt
2 |  10320/LOC |  2016-02-22 17:46:04Z |   \<locations\>\<location href="irods://145.100.58.24:1247/bobZone/home/alice#aliceZone/DataCollection/put1.txt" id="0"/\>\</locations\>
3 |  CHECKSUM  |  2016-02-22 17:46:04Z |   d6eb32081c822ed572b70567826d9d9d
4 |  EUDAT/ROR |  2016-02-22 17:46:04Z  |  846/6e67a674-d98a-11e5-b634-04040a64000c
5 |  EUDAT/PPID |  2016-02-22 17:46:04Z  |  846/6e67a674-d98a-11e5-b634-04040a64000c

The replica contains two extra fields.
*EUDAT/ROR* indicates the original file in the repository of resources.
*EUDAT/PPID* contains the PID to the direct parent.

The ROR-entry is important to verify that the replica is indeed the same as the ROR, which has to be done by integrity checks. Every replica, also a replica of a replica, will inherit this entry. The PPID entry is important to build the linked list of replicas in case replicas are further replicated to other sites.

### Retrieve the PIDs of the replicas

Option 1)

As B2SAFE admin you have access to the PIDs of the parent PID in your iCAT catalogue.

**Exercise** If you already followed the [PID tutorial](https://github.com/chStaiger/B2SAFE-B2STAGE-Training/blob/master/0X-Working-with-PIDs_epicclient.md) write a script to fetch all PIDs of the replicas and check whether original and replica indeed have the same checksum

Option 2)

**Exercise** Same as in Option 1) but use the information the two iCAT catalogues and the function *ichksum*. Tip: you can access the data and the iCAT of bobZone like this:

```sh
ils -L /bobZone/home/alice#aliceZone/DataCollection
imeta ls -d /bobZone/home/alice#aliceZone/DataCollection/put1.txt
```

**Exercise** Replicate the data from bobZone in a different collection in aliceZone, inspect the PID entries and write a script to communicate the whole linked list of PIDs to your irods useri, e.g. as a text file.

