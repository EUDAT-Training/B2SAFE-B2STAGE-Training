# Working with iRODS resources and federations
This part of the tutorial will show how you can work with resources, automate data policies and transfer data across iRODS zones via federations.
This part of the tutorial assumes that you either followed the tuorial [part 02 - for admins](https://github.com/EUDAT-Training/B2SAFE-B2STAGE-Training/blob/master/02-iRODS-handson-admin.md)
or that an irodsadmin has created several resources on the iRODS instance you are working on.

[//]: # "Can or must?"
All commands can be executed as *irods user* from the user interface machine.

## Data and data resources

In the first part of the tutorial you ingested the file *put1.txt* into iRODS using the *demoResc*. With

```sh
ilsresc
```
we can check which other resources are available. E.g.

```
demoResc
globalResc
newResc
replResc:replication
├── storage1
└── storage2
```

To replicate a file *put1.txt* to the *globalResc* and *newResc* resource, execute
```sh
irepl -R globalResc put1.txt
irepl -R newResc put1.txt
```
The option *-R* indicates the resource.

You can list replicas with
```sh
ils -l put1.txt
```

which will yield
```
  alice             0 demoResc           13 2016-02-22.18:06 & put1.txt
  alice             1 globalResc           13 2016-05-05.15:57 & put1.txt
  alice             2 newResc           13 2016-05-05.15:58 & put1.txt
```

Note, that all replicas are numbered. This number can be used to delete replicas:
```sh
itrim -n 1 put1.txt
```

**Exercise** What happens if you replicate the file again to *globalResc*?

**Exercise** What happens if you call *itrim* without the *-n* option?

**Exercise** How can you reduce the number of replicas to 1?

[//]: # "Headers are missing. Is this correct?"
[]()  | []()
------|------
irepl   | Replicate data to a resource
itrim   | Reduce number of replicas
isync   | Replicate to another iRODS zone

## Replicating data between iRODS zones
We can replicate data between our iRODS zone and another iRODS zone. At the other iRODS zone the local user name needs to extended with *#<localzone>*.

First let's have a look at the data under the remote account:
```sh
ils /bobtestZone/home/alice#alicetestZone
```

We can copy data to the remote zone:
```sh
irsync -R demoResc i:/alicetestZone/home/alice/put1.txt \
 i:/bobtestZone/home/alice#alicetestZone/put1.txt
```
You can also directly ingest data into the remote iRODS instance
```sh
irsync -R demoResc put2.txt i:/bobtestZone/home/alice#alicetestZone/put2.txt
```
or fetch data from the remote ionstance and safe it to your device and store it under a different name.
```sh
irsync -R demoResc i:/bobtestZone/home/alice#alicetestZone/put2.txt put3.txt
```

## iRODS rules
iRODS offers the possibility to automate data management processes by creating scripts written in the iRODS rule language.

Save the example rule below in a file called *HelloWorld.r*
```sh
HelloWorld{
    writeLine("stdout", "Hello iRODS world!")
}
input null
output ruleExecOut
```
and execute the rule with
```sh
irule -F testRules/HelloWorld.r
```

You might have realised that the *ls* command just lists subcollections and files in the collection you execute it in. To list all files and collection recursively, we can write a rule.
```sh
recursiveList{
    foreach(*row in SELECT COLL_NAME, DATA_NAME WHERE COLL_NAME like '%home%'){
        *coll = *row.COLL_NAME;
        *data = *row.DATA_NAME;
        writeLine("stdout", "*coll/*data");
    }
    writeLine("stdout", "listing done");
}

input null
output ruleExecOut
```
The '%' works as wild card, variables are denoted by '*'.

### Passing arguments and ouput
HelloWorld{
    if(*name=="<YourName>"){
        writeLine("stdout", "Hello *name!");
        }
    else { writeLine("stdout", "Hello world!"); }
}
INPUT *name="YourName"
OUTPUT ruleExecOut, *name

We can overwrite input parameters by calling the function like this:
```sh
irule -F testRules/HelloWorld.r "*name='Alice'"
```

**Exercise** In the last exercise of the module [01-iRODS-handson-user.md](https://github.com/EUDAT-Training/B2SAFE-B2STAGE-Training/blob/master/01-iRODS-handson-user.md) you needed to combine two queries, one for the data files and one for the collections, to find all items that carry a certain metadata entry. Combine these two queries in one rule.

[]()  | []()
------|------
irule   | Execute a rule
idbug   | Step-by-step execution of a rule
iqstat  | List scheduled rules
iqdel   | Delete a scheduled rule

### The *core.re* and example rules
iRODS provides a default rule base in */etc/irods/core.re*. These rules can be employed and called by your own rules.
More examples how rules can look like are provided in */var/lib/irods/iRODS/clients/icommands/test/rules3.0/*.

In the following example we make use of the *printHello* rule from the *core.re*:
```sh
HelloWorld{
    if(*name=="<YourName>"){
        writeLine("stdout", "Hello *name!");
        }
    else { printHello; }
}
INPUT *name="YourName"
OUTPUT ruleExecOut, *name
```

### Scheduled rules

We can also delay rules and execute rules regularly.
```sh
HelloWorld{
    delay("<PLUSET>1m</PLUSET><EF>5m</EF>"){
        msiWriteRodsLog("Hello World.", *status);
    }
}
INPUT  null
OUTPUT ruleExecOut
```
The function *delay* delayes the execution by 1 minute and restarts the rule automatically every 5 minutes. With *iqstat* we can check the status of the rule.
The output of the rule is written to the rodsLog file in */var/lib/irods/iRODS/server/log/reLog*.

### Microservices
Microservices are small and well-defined functions to perform simple tasks. A list of pre-implemented microservices can be found [here](https://docs.irods.org/master/doxygen/).

**Example** for calling an external python script via the microservice *msiExecCmd*. The rule fetches the help for the python script.
```sh
myTestRule{
        msiExecCmd("epicclient.py", "-h",
                   "null", "null", "null", *Result);
        msiGetStdoutInExecCmdOut(*Result,*Out);
        writeLine("stdout","*Out");
}
INPUT null
OUTPUT ruleExecOut
```
The first argument of *msiExecCmd* is the actual command. In that case the python script begins with the hash-bang *#!/usr/bin/env python* which makes it executable. The second argument is a list of parameters and the last stores the output of the executed command.
**Note** that all commands that you call need to be located in *iRODS/server/bin/cmd*. To add new commands to that folder you need *sudo* rights on the iRODS server.

### Exercise
Write a rule that periodically checks whether new data is ingested into a certain collection and automatically replicate the data to a dedicated storage resource. You can make use of delayed iRODS rules. Some microservices can be of help.
