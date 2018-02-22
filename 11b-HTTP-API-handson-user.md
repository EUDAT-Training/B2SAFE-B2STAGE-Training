# HTTP API - Hands-on
This document will show you how to use the HTTP API. It will explain endpoints, fuctionality and the connection between the API and B2SAFE from the user's side.

## Prerequisites
You need to have an own running [test instance](11a-Setup-HTTP-API.md) or access to the [EUDAT prototype](https://github.com/EUDAT-B2STAGE/http-api/blob/master/docs/prototype.md).
We will work in the following with *curl* commands. All these commands can also be tested through the swagger interface of the HTTP API.
The swagger interface for the prototype can be reached under `http://petstore.swagger.io/?url=https://b2stage-test.cineca.it/api/specs&docExpansion`. If you are working with your own test instance please go to `http://IP or FQDN/?docExpansion` and replace 'localhost' in the serach bar with '\<IP or FQDN:8080\>'.

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

In the **swagger interface** please use the endpoints *Authentication/post/b2safeproxy* or *Authentication/post/auth/login* respectively.
If you want to continue working with swagger you have to click the lock button next to *authentication* and provide (copy paste) your token there. Now the token is saved for all other operations through the swagger interface. 

In the following sections we will have a lok at how to upload data to different endpoints and what that means on iRODS level.

## The *registered* enpoint
The endpoint called *registered* corresponds to a domain in iRODS, where each data object carries a persistent identifier (PID). The HTTP API itself does **not** assign the PID, this is done by iRODS rules, e.g. the eventhooks we decribed in the previous section.
By default, whe you enabled your iRODS instance with the HTTP API but did not install B2SAFE or configured these eventhooks **no PIDs will be assigned**.
No matter whether PIDs are assigned or not you can upload, download, change or delete data under this endpoint.
When you are working with swagger please use the endpoint *registered* and choose the respective operation. (**NOTE**  curl commands in swagger are not formed correctly, status Feb 2018 --> will currently not work.)

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

### Downloading data
We can now also download the file:

```sh
mkdir API-downloads
curl -o API-downloads/downloaded.png -H "Authorization: Bearer $TOKEN" $SERVER/api/registered/tempZone/home/alice/upload/my.png?download=true
```
Open the file under *API-downloads*.

### Deleting files
The HTTP API also supports the deletion of files and collections. However, when using PIDs, deletion should only take place according to the data management and PID policies. Be careful of what you delete!

Let us try to delete the whole collection:
```sh
curl -X DELETE -H "Authorization: Bearer $TOKEN" \
  $SERVER/api/registered/tempZone/home/alice/upload
```
You will receive an answer
```
"status": 400
"Directory is not empty"
```

If we first delete the object
```sh
curl -X DELETE -H "Authorization: Bearer $TOKEN" \
  $SERVER/api/registered/tempZone/home/alice/upload/my.png
```
And then again the collection
```sh
curl -X DELETE -H "Authorization: Bearer $TOKEN" \
  $SERVER/api/registered/tempZone/home/alice/upload
```

all data will be gone and we have again our empty home collection. Note, if you assigned PIDs to data and collections, the PIDs will still exist and possibly point to a none existing location.

## The *publish* endpoint
The HTTP API offers to make data publicly available. It creates a landing page which can be accessed conveniently with a browser and from which files can be downloaded.
Please note, the *publish* endpoint does not offer to publish data like the B2SAHRE service does, where the data ownership is transferred to a different party and the data is handled according to B2SHARE data policies. 
Here, the user will still be the owner of the file and has full command over it, i.e. a user can remove his/her publicly accessible data. 

We will demonstrate the usage of the *publish* endpoint on the B2STAGE HTTP API test instance. If you want to enable your own environment with that endpoint please make sure you:
- Checked out the branch 1.0.2 or higher
- Uncommented '# IRODS_ANONYMOUS: 1' in *projects/b2stage/project_configuration.yaml*, around line 89

### Make data publicly accessible
We can only put data to the *publish* endpoint which is already uploaded to one of the other endpoints e.g. the *registered* endpoint. Basically, we are not creating a new data object, we merely tag the existing data. The B2STAGE HTTP API then allows the iRODS 'anonymous' user access to the data file.

- Upload to *registered*
 ```sh
 curl -X PUT -H "Authorization: Bearer $TOKEN" -F file=@/Users/christines/my.png \
 $SERVER/api/registered/cinecaDMPZone1/home/<user>/my.png
 ```

- Make publicly accessible
 ```sh
 curl -X PUT -H "Authorization: Bearer $TOKEN" $SERVER/api/publish/cinecaDMPZone1/home/<user>/my.png
 ```
 You will receive an answer:
 
 ```sh
   "Response": {
    "data": {
      "published": true
    },
    "errors": null
 ```
 
 Now others can access the landing page of the data by 
 
 `https://b2stage-test.cineca.it/api/public/cinecaDMPZone1/home/<user>/my.png`
 
- Withdraw public access
 As the owner of the publicly accessible file you can withdraw the data:
 ```sh
  curl -X DELETE -H "Authorization: Bearer $TOKEN" $SERVER/api/publish/cinecaDMPZone1/home/<user>/my.png
 ```
 which will return:
 ```sh
 "Response": {
    "data": {
      "published": false
    },
    "errors": null
  }
 ```
 The link from above is not valid any longer. However, the data under the *registered* endpoint is still there and its B2SAFE metadata can be listed:
 
 ```sh
 curl -H "Authorization: Bearer $TOKEN" $SERVER/api/registered/cinecaDMPZone1/home/<user>/my.png
 ```

### Some notes
- The HTTP API does not yet support making iRODS collections publicly accessible.
- When data is removed from the *registered* endpoint, it will also no longer be available under the *publish* endpoint.

