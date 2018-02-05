# HTTP API - Hands-on
This document will show you how to use the HTTP API. It will explain endpoints, fuctionality and the connection between the API and 2SAFE from the user's side.

## Prerequisites
You need to have an own running [test instance](11a-Setup-HTTP-API.md) or access to the [EUDAT prototype](https://github.com/EUDAT-B2STAGE/http-api/blob/master/docs/prototype.md).
We will work in the following with *curl* commands.

## API token
The API supports two types of users: 1) B2ACCESS users and 2) native iRODS users. They receive their token via different authorisation endpoints.
If you use your own test instance there is a *guest* user which which is authenticated via B2ACCESS. For this user you can get the token via the */auth/login* endpoint.

```sh
curl -X POST http://<IP or FQDN>:8080/auth/login -d '{"username":"user@nomail.org","password":"test"}'
```

When you have the credentials for a native iRODS user, you can receive your token from the */auth/b2safeproxy* endpoint with your iRODS password.

```sh
curl -X POST http://<IP or FQDN>:8080/auth/b2safeproxy -d '{"username":"alice","password":"<safepw>"}'
```

You will receive a json file with the token:
```sh
{
  "Meta": {
    "data_type": "<class 'dict'>",
    "elements": 3,
    "errors": 0,
    "status": 200
  },
  "Response": {
    "data": {
      "b2safe_home": "/tempZone/home/alice",
      "b2safe_user": "alice",
      "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoiOGZmNTM4"
    },
    "errors": null
  }
}
```
Now create two shell variables to reduce typing:

```sh
TOKEN='<token>'
SERVER='http://<IP or FQDN>:8080'
```

In the following sections we will have a lok at how to upload data to different endpoints and what that means on iRODS level.

## The *registered* enpoint
The endpoint called *registered* corresponds to a domain in iRODS, where each data object carries a persistent identifier (PID). The HTTP API itself does **not** assign the PID, this is done by iRODS rules, e.g. the eventhooks we decribed in the previous section.
By default, whe you enabled your iRODS instance with the HTTP API but did not install B2SAFE or configured these eventhooks **no PIDs will be assigned**.
No matter whether PIDs are assigned or not you can upload, download, change or detele data under this endpoint.

### iRODS collection listing
We will first list our iRODS collection:

```
curl -H "Authorization: Bearer $TOKEN" $SERVER/api/registered/tempZone/home/alice

{
  "Meta": {
    "data_type": "<class 'list'>",
    "elements": 0,
    "errors": 0,
    "status": 200
  },
  "Response": {
    "data": [],
    "errors": null
  }
}
```
We see that our data collection in iRODS is empty.

### Creating a new collection
Now we can create a new collection in the domain registered:

```
curl -X POST -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
-d '{"path":"/tempZone/home/alice/upload"}' $SERVER/api/registered

{
  "Meta": {
    "data_type": "<class 'dict'>",
    "elements": 3,
    "errors": 0,
    "status": 200
  },
  "Response": {
    "data": {
      "link": "http://localhost:8080/api/registered/tempZone/home/alice/upload",
      "location": "irods://rodserver.dockerized.io/tempZone/home/alice/upload",
      "path": "/tempZone/home/alice/upload"
    },
    "errors": null
  }
}
```
This command creates new collection without any metadata. If we issue the same command again, i.e. try to overwrite existing collections and data, the API will inform us

```sh
"errors": 1,
    "status": 400
"B2SAFE: Irods collection already exists"
```

### Uploading a file
We can no upload a file to the new collection:

```sh
curl -X PUT -H "Authorization: Bearer $TOKEN" -F file=@/Users/christines/my.png \
$SERVER/api/registered/tempZone/home/alice/upload/my.png

{
  "Meta": {
    "data_type": "<class 'dict'>",
    "elements": 6,
    "errors": 0,
    "status": 200
  },
  "Response": {
    "data": {
      "PID": null,
      "checksum": null,
      "filename": "my.png",
      "link": "http://localhost:8080/api/registered/tempZone/home/alice/upload/my.png",
      "location": "irods://rodserver.dockerized.io/tempZone/home/alice/upload/my.png",
      "path": "/tempZone/home/alice/upload/my.png"
    },
    "errors": null
  }
}
```

Note, the HTTP API does not support upload of directories or recursive uploads.

Now we can again list the collection *upload*:

```sh
curl -H "Authorization: Bearer $TOKEN" $SERVER/api/registered/tempZone/home/alice/upload

{
  "Meta": {
    "data_type": "<class 'list'>",
    "elements": 1,
    "errors": 0,
    "status": 200
  },
  "Response": {
    "data": [
      {
        "my.png": {
          "dataobject": "my.png",
          "link": "http://localhost:8080/api/registered/tempZone/home/alice/upload/my.png",
          "location": "irods://rodserver.dockerized.io/tempZone/home/alice/upload",
          "metadata": {
            "EUDAT/FIO": null,
            "EUDAT/FIXED_CONTENT": null,
            "EUDAT/PARENT": null,
            "EUDAT/REPLICA": null,
            "EUDAT/ROR": null,
            "PID": null,
            "checksum": null,
            "name": "my.png",
            "object_type": "dataobject"
          },
          "path": "/tempZone/home/alice/upload"
        }
      }
    ],
    "errors": null
  }
}
```

The HTTP API lists all data in the collection, here our file we upoaded previously. Additionally uder the endpoint *registered*, the command lists as metadata all B2SAFE-specific metadata fields. Since B2SAFE is not installed or has not been called on this data object yet all of the metadata fields are empty.







