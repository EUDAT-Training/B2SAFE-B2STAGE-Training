# Setting up an iRODS federation
This hands-on takes you through the configuration steps necessary to set up an iRODS federation.

## Prerequisites
Two iRODS 4.1 zones.

## Configuring the iRODS federation
### Creating remote zones and users
Assume we have two iRODS servers *aliceZone* with *alice* as iRODS admin and *bobZone* with *bob* as iRODS admin.
- We need to create remote zones on the respective machines, i.e. on *aliceZone* we need to create a remote zone for *bobZone* and vice versa. On *aliceZone* do
[//]: # "I presume that the full hostname or ipaddress is that of alice."
[//]: # "I would make that a bit more clear here."
```sh
iadmin mkzone bobZone remote <full hostname or ipadress>:1247
```
Note that you cannot rename *bobZone*, the remote zone name needs to be exactly the same zone name as on the iRODS server you would like to federate with. On *bobZone* do
```sh
iadmin mkzone aliceZone remote <full hostname or ipadress>:1247
```

[//]: # "rodsuser is not yet explained."
- Next we need to grant access to *alice* on *bobZone* as **rodsuser**
```sh
iadmin mkuser alice#aliceZone rodsuser
```
- And on *aliceZone* we need make *bob* known as a user
```sh
iadmin mkuser bob#bobZone rodsuser
```
The '#' denotes the zone where the user *alice* is known and authenticated.
**rodsuser** gives alice user rights. With
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
```
You will find all required information in the server_config.json on *aliceZone*.
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

- In some cases you will also have edit the /etc/irods/hosts_config.json. This is the case if you encounter the following error after editing the server_config.json you will have to go through another step.
```sh
bob@irods4:~$ ils /aliceZone/home/bob#bobZone
ERROR: connectToRhost: error returned from host localhost status = -38000 status = -38000 SYS_AGENT_INIT_ERR
ERROR: _rcConnect: connectToRhost error, server on localhost:1247 is probably down status = -38000 SYS_AGENT_INIT_ERR
```
Open the hosts_config.json and enter on *aliceZone* the addresses of your local zone and the remote zone (*bobZone*) (please remove the comments):
[//]: # "I would not use those ip addresses. Better using a private ip address."
[//]: # "You can pick anyone from this range: 10.0.0.0 - 10.255.255.255"
[//]: # "Same for the FQDN. It is better not use surfsara.nl, example.org"
```sh
{
    "host_entries": [
{
            "address_type" : "remote",
            "addresses" : [
                   {"address" : "145.100.58.24"}, #ip address
                   {"address" : "iRODS4-bob.eudat-sara.vm.surfsara.nl"}, #full server name
                   {"address" : "irods4.bob"} #hostname
             ]
        },
        {
            "address_type" : "local",
            "addresses" : [
                   {"address" : "145.100.58.12"},
                   {"address" : "iRODS4-alice.eudat-sara.vm.surfsara.nl"},
                   {"address" : "iRODS4.alice"}
             ]
        }
]
}
```
Do the same on *bobZone* and restart the iRODS servers as *root*.

```sh
service irods restart
```

After logging into irods again on *bobZone* you are now able to list your folders and files on *aliceZone*:
```sh
iinit
bob@irods4:~$ ils /aliceZone/home/bob#bobZone
/aliceZone/home/bob#bobZone:
```

We explain [HERE](https://github.com/EUDAT-Training/B2SAFE-B2STAGE-Training/blob/master/05-iRODS-advanced-users.md) the basics of how you can manage data across different iRODS zones.
In our [B2SAFE-HANDS-ON](https://github.com/EUDAT-Training/B2SAFE-B2STAGE-Training/blob/master/06-B2SAFE-handson.md) we illustrate how to employ policies for data management in iRODS federations.
