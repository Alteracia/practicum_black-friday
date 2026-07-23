# pymongo-api

## Как запустить

Запускаем mongodb и приложение

```shell
docker compose up -d
```

## Инициализация

### Инициализируем конфиг сервер
```shell
docker compose exec -T configSrv mongosh --port 27017
> rs.initiate(
  {
    _id : "config_server",
       configsvr: true,
    members: [
      { _id : 0, host : "configSrv:27017" }
    ]
  }
);
> exit();
```
```shell
./scripts/mongo-sharding-init_1.sh
```

### Инициализируем шарды:
```shell
docker compose exec -T shard1-1 mongosh --port 27018

> rs.initiate(
    {
      _id : "shard1",
      members: [
          { _id : 0, host : "shard1-1:27018" },
          { _id : 1, host : "shard1-2:27018" },
          { _id : 2, host : "shard1-3:27018" }
      ]
    }
);
> exit();
```
```shell
./scripts/mongo-sharding-init_2.sh
```

```shell
docker compose exec -T shard2-1 mongosh --port 27018

> rs.initiate(
    {
      _id : "shard2",
      members: [
          { _id : 0, host : "shard2-1:27018" },
          { _id : 1, host : "shard2-2:27018" },
          { _id : 2, host : "shard2-3:27018" }
      ]
    }
  );
> exit();
```
```shell
./scripts/mongo-sharding-init_3.sh
```

### Инициализируем роутер и заполняем документами
```shell
docker compose exec -T mongos_router mongosh --port 27024

> sh.addShard( "shard1/shard1-1:27018");
> sh.addShard( "shard2/shard2-1:27018");

> sh.enableSharding("somedb");
> sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } )

> use somedb

> for(var i = 0; i < 1000; i++) db.helloDoc.insert({age:i, name:"ly"+i})
> exit();
```
```shell
./scripts/mongo-sharding-init_4.sh
```

## Проверка

### В браузере
Откройте в браузере http://localhost:8080

### Базы напрямую
Проверка роутера
```shell
docker exec -it mongos_router mongosh --port 27024
> use somedb
> db.helloDoc.countDocuments()
> exit(); 
```
```shell
./scripts/mongo-sharding-info_1.sh
```
```
1000
```

Проверка shard1
```shell
docker exec -it shard1-1 mongosh --port 27018
> use somedb
> db.helloDoc.countDocuments()
> exit(); 
```
```shell
./scripts/mongo-sharding-info_2.sh
```

```
492
```

Проверка shard2
```shell
docker exec -it shard2-1 mongosh --port 27018
> use somedb
> db.helloDoc.countDocuments()
> exit(); 
```
```shell
./scripts/mongo-sharding-info_3.sh
```

```
508
```