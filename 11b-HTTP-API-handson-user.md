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
      "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoiOGZmNTM4ZDItOWYzNy00MzMwLTlhOWEtMmZkY2U4ZGExZTlhIiwianRpIjoiNDQyZjNmMjUtY2JjMS00MjczLWFmM2YtODIyNjQyNTBmYzBiIn0.-FaDPLvHGTzRjGwTQU17jjB4ctCwVQ4ZkcDMwyyiNII"
    },
    "errors": null
  }
}
```
Now create two shell variables to reduce typing:

```
TOKEN='<token>'
SERVER='http://<IP or FQDN>:8080'
```

