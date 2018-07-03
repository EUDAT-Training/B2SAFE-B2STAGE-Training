# Setting up an iRODS federation
This hands-on takes you through the configuration steps necessary to set up an iRODS federation.

## Prerequisites
Two iRODS 4.1 zones e.g. *aliceZone* and *bobZone*. You will need *rodsadmin* rights on both zones.

## Configuring the iRODS federation
### Creating remote zones and users
Assume we have two iRODS servers *aliceZone* with *alice* as iRODS admin and *bobZone* with *bob* as iRODS admin.

- We need to create remote zones on the respective machines, i.e. on *aliceZone* we need to create a remote zone for *bobZone* and vice versa. On *aliceZone* do
```sh
iadmin mkzone bobZone remote <full hostname or ipadress of bob>:1247
```
Note that you cannot rename *bobZone*, the remote zone name needs to be exactly the same zone name 
as on the iRODS server you would like to federate with. On *bobZone* do
```sh
iadmin mkzone aliceZone remote <full hostname or ipadress of alice>:1247
```

- Next we need to grant access to *alice* on *bobZone* as *rodsuser*
```sh
iadmin mkuser alice#aliceZone rodsuser
```

- And on *aliceZone* we need make *bob* known as a user
```sh
iadmin mkuser bob#bobZone rodsuser
```
 The '#' denotes the zone where the user *alice* is known and authenticated.
 *rodsuser* gives alice user rights. With
```sh
iadmin lt user_type
```
 you can check which other user types are known in iRODS.

- However, this is not enough to set up the federation. If you now try to have a look into *bob*'s folder on *aliceZone* you receive the following error:
```sh
bob@irods4:~$ ils /aliceZone
ERROR: rcObjStat of /aliceZone failed status = -913000 REMOTE_SERVER_SID_NOT_DEFINED
```

### Editing the config files
- To make both sites known to each other and to authenticate we need to edit the field 'federations' in /etc/irods/server_config.json
 On *bobZone* insert:
```sh
"federation": [
        {
        "icat_host": "ipadress or full hostname",
        "zone_name": "aliceZone",
        "zone_key": "alice_zone_key",
        "negotiation_key": "TEMPORARY_32byte_negotiation_key"
    }
],
```

- You will find all required information in the server_config.json on *aliceZone*.
 On *aliceZone* insert:
```sh
"federation": [
        {
        "icat_host": "ipadress or full hostname",
        "zone_name": "bobZone",
        "zone_key": "bob_zone_key",
        "negotiation_key": "TEMPORARY_32byte_negotiation_key"
    }
]
```

- In some cases you will also have to edit the /etc/irods/hosts_config.json. 
 This is the case if you encounter the following error after editing the server_config.json you will have to go through another step.
```sh
bob@irods4:~$ ils /aliceZone/home/bob#bobZone
ERROR: connectToRhost: error returned from host localhost status = -38000 status = -38000 SYS_AGENT_INIT_ERR
ERROR: _rcConnect: connectToRhost error, server on localhost:1247 is probably down status = -38000 SYS_AGENT_INIT_ERR
```
 Open the hosts_config.json and enter on *aliceZone* the addresses of your local zone and the remote zone (*bobZone*) (please remove the comments):
```sh
{
    "host_entries": [
{
            "address_type" : "remote",
            "addresses" : [
                   {"address" : "bob.ipa.dddr.ess"}, #ip address
                   {"address" : "<fully qualified hostname>"}, #full server name
                   {"address" : "<localhost>"} #hostname
             ]
        },
        {
            "address_type" : "local",
            "addresses" : [
                   {"address" : "ali.cei.pad.ress"},
                   {"address" : "<fully qualified hostname>"},
                   {"address" : "<localhost>"}
             ]
        }
]
}
```
 Do the same on *bobZone* and restart the iRODS servers as *root*.

```sh
service irods restart
```

### Final check

After logging into irods again on *bobZone* you are now able to list your folders and files on *aliceZone*:
```sh
iinit
bob@irods4:~$ ils /aliceZone/home/bob#bobZone
/aliceZone/home/bob#bobZone:
```

We explain [HERE](https://github.com/EUDAT-Training/B2SAFE-B2STAGE-Training/blob/master/05-iRODS-advanced-users.md) the basics of how you can manage data across different iRODS zones.
In our [B2SAFE-HANDS-ON](https://github.com/EUDAT-Training/B2SAFE-B2STAGE-Training/blob/master/06-B2SAFE-handson.md) we illustrate how to employ policies for data management in iRODS federations.

## Remarks
1) In some cases accessing the federated zone still hangs. If you find the line `pid:2528 ERROR: _rcConnect: setRhostInfo error, irodsHost is probably not set correctly status = -303000 USER_RODS_HOSTNAME_ERR` in your iRODS log files, then try mapping server names to FQDN or IP in the */etc/hosts* file.

[]()|[]()|[]()
----|----|----
[Previous](03-install-B2SAFE.md)|[Index](https://github.com/EUDAT-Training/B2SAFE-B2STAGE-Training)  | [Next](05-iRODS-advanced-users.md)
