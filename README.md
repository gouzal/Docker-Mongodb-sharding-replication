# docker mongodb sharding/replication
[![ForTheBadge built-with-love](http://ForTheBadge.com/images/badges/built-with-love.svg)](https://GitHub.com/Naereen/)
This article will help you to configure and run simple MongoDB Sharding/Replication cluster based on Docker.
## Topology
Topology for our super simple cluster base on using:
* 1 node as config server
* 1 node as router (mongos)
* 3 nodes as shards (ReplicaSet)
* 9 nodes for Replication (include 3 nodes as Arbiter)

## Download MongoDB (4.0.8 = latest) Image for Docker
```sh
docker pull mongo
```
## Docker network
Create docker net for our cluster:
```sh
docker network create --driver bridge xnet
```
## Run config node:
```sh
docker run -it -d  --net=xnet -p 27015:27015 --hostname "config0" --name config0 mongo:latest --port 27015 --replSet config0 --configsvr
```

```sh
docker exec -it -d config0 mongo --port 27015 --eval '
rs.initiate( {
   _id: "config0",
   configsvr: true,
   version: 1,
   members: [ { _id: 0, host: "config0:27015" } ]
} )
'
```
## Run Router (mongos) node:
```sh
docker run -it --net=xnet -d -p 27017:27017 --hostname "router0" --name "router0" mongo:latest mongos --port 27017  --configdb config0:27015
```
```sh
docker exec -it -d config0 mongo --port 27015 --eval '
rs.initiate( {
   _id: "config0",
   configsvr: true,
   version: 1,
   members: [ { _id: 0, host: "config0:27015" } ]
} ) '
```

## Shard 0 (first ReplicaSet):
```sh
docker run -d --net=xnet --name shard00 --hostname="shard00" -p 27022:27022 mongo:latest --replSet "rs0" --port 27022 --shardsvr
```
```sh
docker exec -it -d shard00 mongo --port 27022 --eval '
rs.initiate({ _id : "rs0", members: [ { _id : 0,arbiterOnly : true, host : "shard00:27022" } ] })
'
```
```sh
docker run -d --net=xnet --name shard01 --hostname="shard01" -p 27023:27023 mongo:latest --replSet "rs0" --port 27023 --shardsvr
```
```sh
docker exec -it -d shard01 mongo --port 27023 --eval '
rs.initiate({ _id : "rs0", members: [ { _id : 1, host : "shard01:27023" } ] })
'
```
```sh
docker run -d --net=xnet --name shard02 --hostname="shard02" -p 27024:27024 mongo:latest --replSet "rs0" --port 27024 --shardsvr
```
```sh
docker exec -it -d shard02 mongo --port 27024 --eval '
rs.initiate({ _id : "rs0", members: [ { _id : 2, host : "shard01:27024" } ] })
'
```

## Shard 1 (second ReplicaSet):
```sh
docker run -d --net=xnet --name shard10 --hostname="shard10" -p 27025:27025 mongo:latest --replSet "rs1" --port 27025 --shardsvr
```
```sh
docker exec -it -d shard10 mongo --port 27025 --eval '
rs.initiate({ _id : "rs1", members: [ { _id : 3,arbiterOnly : true, host : "shard10:27025" } ] })
'
```
```sh
docker run -d --net=xnet --name shard11 --hostname="shard11" -p 27026:27026 mongo:latest --replSet "rs1" --port 27026 --shardsvr
```
```sh
docker exec -it -d shard11 mongo --port 27026 --eval '
rs.initiate({ _id : "rs1", members: [ { _id : 4, host : "shard11:27026" } ] })
'
```
```sh
docker run -d --net=xnet --name shard12 --hostname="shard12" -p 27027:27027 mongo:latest --replSet "rs1" --port 27027 --shardsvr
```
```sh
docker exec -it -d shard12 mongo --port 27027 --eval '
rs.initiate({ _id : "rs1", members: [ { _id : 5, host : "shard12:27027" } ] })
'
```

## Shard 3 (Third ReplicaSet)
```sh
docker run -d --net=xnet --name shard20 --hostname="shard20" -p 27028:27028 mongo:latest --replSet "rs2" --port 27028 --shardsvr 
```
```sh
docker exec -it -d shard10 mongo --port 27025 --eval '
rs.initiate({ _id : "rs2", members: [ { _id : 6,arbiterOnly : true, host : "shard20:27025" } ] })
'
```
```sh
docker run -d --net=xnet --name shard21 --hostname="shard21" -p 27029:27029 mongo:latest --replSet "rs2" --port 27029 --shardsvr
```
```sh
docker exec -it -d shard21 mongo --port 27029 --eval '
rs.initiate({ _id : "rs2", members: [ { _id : 7, host : "shard21:27029" } ] })
'
```
```sh
docker run -d --net=xnet --name shard22 --hostname="shard22" -p 27030:27030 mongo:latest --replSet "rs2" --port 27030 --shardsvr
```
```sh
docker exec -it -d shard22 mongo --port 27030 --eval '
rs.initiate({ _id : "rs2", members: [ { _id : 8, host : "shard22:27030" } ] })
'
```

## Add shards to Router (mongos) and create database

```sh
docker exec -it  router0 mongo --port 27017 --eval '
    sh.addShard("rs0/shard00:27022");
	sh.addShard("rs0/shard01:27023");
	sh.addShard("rs0/shard02:27024");
	sh.addShard("rs1/shard10:27025");
	sh.addShard("rs1/shard11:27026");
	sh.addShard("rs1/shard12:27027");
	sh.addShard("rs2/shard20:27028");
	sh.addShard("rs2/shard21:27029");
	sh.addShard("rs2/shard22:27030");

    sh.enableSharding("mydb");
    sh.shardCollection("mydb.test_collection", {"tag": 1});
    sh.status();  '

```

License
----

[![GitHub license](https://img.shields.io/github/license/Naereen/StrapDown.js.svg)](https://github.com/Naereen/StrapDown.js/blob/master/LICENSE)
[![Open Source Love svg1](https://badges.frapsoft.com/os/v1/open-source.svg?v=103)](https://github.com/ellerbrock/open-source-badges/)
[![saythanks](https://img.shields.io/badge/say-thanks-ff69b4.svg)](https://saythanks.io/to/kennethreitz)


**Free Software, Hell Yeah!**
