# Redis example with Jedis

Ran a redis container on windows using Docker (just make sure Docker Engine is running before executing the below command) 
```
docker run --name some-redis --publish=6379:6379 -d redis
```

!["Redis Container"](redis-container.png?raw=true)

## Jedis
```
    <dependency>
  		<groupId>redis.clients</groupId>
  		<artifactId>jedis</artifactId>
  		<version>5.2.0</version>
  	</dependency>
```
