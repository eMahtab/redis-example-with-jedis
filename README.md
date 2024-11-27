# Redis example with Jedis

Ran a redis container on windows using Docker (just make sure Docker Engine is running before executing the below command) 
```
docker run --name some-redis --publish=6379:6379 -d redis
```

!["Redis Container"](redis-container.png?raw=true)

## Jedis
```xml
<dependency>
   <groupId>redis.clients</groupId>
   <artifactId>jedis</artifactId>
   <version>5.2.0</version>
</dependency>
```

## Example 1 :
```java
import redis.clients.jedis.Jedis;

public class Test {
    public static void main(String[] args) {
        try (Jedis jedis = new Jedis("localhost", 6379)) {
            String res1 = jedis.set("user:1", "Mahtab Alam");
            System.out.println(res1); // OK

            String res2 = jedis.get("user:1");
            System.out.println(res2); // Mahtab Alam

            jedis.setex("active:partition", 10, "edf833-hju87iols-kla12");
            Thread.sleep(6000);
            System.out.println("Active Partition :" + jedis.get("active:partition"));
            Thread.sleep(5000);
            System.out.println("Active Partition :" + jedis.get("active:partition"));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### Code Execution Output :
```
OK
Mahtab Alam
Active Partition :edf833-hju87iols-kla12
Active Partition :null
```
