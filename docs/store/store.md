---
layout: default
title: Store
nav_order: 3
has_children: true
permalink: /docs/store
---

- [Haaukins Store](#haaukins-store)
  - [Production usage](#production-usage)
  - [Configuration](#configuration)
    - [Environment File](#environment-file)
    - [Configuration file](#configuration-file)
  - [Docker compose](#docker-compose)
  - [Run](#run)
  
# Haaukins Store 

[Haaukins Store](https://github.com/aau-network-security/haaukins-store){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 } 


Haaukins store is internally used for managing information about events and teams which are exists in Hauukins. With gRPC communication, [Haaukins](https://github.com/aau-network-security/haaukins) is able to get/post information into haaukins store, although we have store folder in Haaukins repo, we are retrieving and updating information through Haaukins store. The one which is exists on Haaukins is just used for caching purposes. 

However, we have some local data which consists of configuration files which are fetched and updated directly from host for Haaukins. They are namely; 
 
- `config.yml` : This is main file to run Haaukins daemon, it specifies all necesseary information regarding to private registries, users, frontends and exercises file location. 
- `exercises.yml`: This file includes information about existing challenges in Haaukins environment. Since it is too strucctured, it was not ok to combine into Haaukins store, however we are thinking to replace it.  
- `frontends.yml` : Provides overall information about frontend which is used in Haaukins, frontends are instances in this context, like `Kali`, `Parrot`, `Ubuntu`.
- `users.yml` : Have information about users who have access to administrator side of Haaukins. 


## Production usage

Docker image of haaukins store could be used in any docker compose file if environment variables provided correctly. When using in production, you can specify image address instead of building it fromm source code. 

Haaukins store image with recent changes will be available at docker hub, with released tag. 

No need to clone the repository, make sure that `.env`  and `config.yml` files are set correctly. 

Steps to run it in production: 

 - Make sure you have configured `.env` and `config.yml` according to the instructions in [configuration](#configuration) section.
```bash 
  curl -o docker-compose.yml https://raw.githubusercontent.com/aau-network-security/haaukins-store/master/docker-compose.yml
```

 - Change [`build: .`](https://github.com/aau-network-security/haaukins-store/blob/d41b09f9aecbec5bb0d6cb687ba9bb6bb8e24378/docker-compose.yml#L7) into  `image: aaunetworksecurity/haaukins-store:<release-tag>` 
 - If everything works as expected, you have recent changes in your server. 
 
 
##  Configuration 

Haaukins store uses two crucial configuration files which are namely, [`.env`](#environment-file) for [docker-compose.yml](https://github.com/aau-network-security/haaukins-store/blob/master/docker-compose.yml) and `config.yml` for retrieving some information in gRPC server side. 


### Environment File

Here is the information which should be included into `.env` file: 

```text
CERTS_PATH=/scratch/configs/certs
CONFIG_PATH=/scratch/configs/config.yml
POSTGRES_DB=exampledb
# POSTGRES_HOST_AUTH_METHOD="trust"
POSTGRES_PASSWORD=exammplepassword

```

- `CERTS_PATH` : Should be provided if TLS is enabled  and certificates should be valid for provided host in [config.yml](#config) file. 
- `CONFIG_PATH`: Place to your `config.yml` file which is mount in `docker-compose.yml` file
- `POSTGRES_DB`: This is the database information that you have provided in [config.yml](#config) file. 
- `POSTGRES_HOST_AUTH_METHOD`: This parameter could be used for local developments however it is NOT recommended, because it eliminates authentication (no password for postgres connection)
- `POSTGRES_PASSWORD`: Recommended way of running  haaukins store, should be same with the one in [config.yml](#config) file. 
- `POSTGRES_USER`: Could be useful to have, should be same with `db.user` parameter in [config.yml](#config) file

Note that there could be cases where password is not required, in those cases `POSTGRES_HOST_AUTH_METHOD` could be used. However when you are using it, you do NOT need to provide `POSTGRES_PASSWORD`. 

### Configuration file 

Example configuration file to run haaukins store without any error. 

```yaml
host: localhost:50051
auth-key: development-auth-key
signin-key: development-signin-key
db:
  host: postgres-db 
  user: postgres
  pass: postgres
  db_name: dummydb
  db_port: 5432
tls:
  enabled: false
  certfile: ./tests/certs/localhost_50051.crt
  certkey: ./tests/certs/localhost_50051.key
  cafile: ./tests/
```

- `host`: It is gRPC server host address which means that the server, that will be run through docker compose,  will run on that address.
- `auth-key`: This is authentication key between gRPC server and client, which means that when haaukins store client is used, `auth-key` should match between server and client. 
- `signin-key`: Similar rule applies as `auth-key`, signing  key should also match to be able to use gRPC calls.
- `db.host` : This is the host name under db configuration, since haaukins store is using docker compose and we are running server with docker compose, it is ok to use service name as database host.
- `db.user`: As name declares, it is database user. 
- `db_name`: Database name, which should be same with the one in your [`.env`](#environment-file)
- `db_port`: It is the port to lookup by server which will be build during `docker-compose run -d`
- `tls`: This consists of some information regarding to your certificates paths, if `tls.enabled` is true which means that you are preferring to use secure communication between server and client. 


## Docker compose 

Docker compose file is defining how services will communicate and how they will be called when they run. The defined services which are defined in docker-compose.yml file might change during time. 
However, the changes will be written here, currently it uses port 5432 for postgres and port 50051 for gRPC server communication. 
Within `docker-compose.yml`, pgadmin4 service is disabled because we decided to use our own tool instead of that one. It could be enabled if we desire at some point. 


## Run

Haaukins store could be run by ; 

- `docker-compose run -d` : will build and run images which are defined in `docker-compose.yml` file IF this is your first time to run `docker-compose.yml` file.

Could be re-build and run by ; 

- `docker-compose run -d --build` : If you performed some changes in source code, you need to add  `--build` flag to `docker-compose run -d`. 

Could be removed by; 

 ```bash 
  docker-compose down --remove-orphans
 ```


