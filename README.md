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

## Example 2 :
```java
import redis.clients.jedis.Jedis;
import java.util.Map;

public class Hashes {
    public static void main(String[] args) {
        try (Jedis jedis = new Jedis("localhost", 6379)) {
            String hashKey = "user:1001";
            // Add fields to the hash using HSET
            jedis.hset(hashKey, "name", "Mahtab Alam");
            jedis.hset(hashKey, "age", "31");
            jedis.hset(hashKey, "city", "Bangalore");

            // Retrieve a specific field using HGET
            String name = jedis.hget(hashKey, "name");
            System.out.println("Name: " + name);

            // Retrieve all fields and values using HGETALL
            Map<String, String> userDetails = jedis.hgetAll(hashKey);
            System.out.println("User Details:");
            userDetails.forEach((field, value) -> System.out.println(field + ": " + value));

            // Increment a numeric field using HINCRBY
            jedis.hincrBy(hashKey, "age", 1);
            System.out.println("Updated Age: " + jedis.hget(hashKey, "age"));

            // HKEYS returns a Set containing all keys in a hash
            System.out.println("Fields: " + jedis.hkeys(hashKey));

            // HVALS returns a List containing all values in a hash
            System.out.println("Values: " + jedis.hvals(hashKey));

            // Delete a field using HDEL
            jedis.hdel(hashKey, "city");
            System.out.println("After deleting 'city', fields: " + jedis.hkeys(hashKey));

            // Check if a field exists using HEXISTS
            boolean hasCity = jedis.hexists(hashKey, "city");
            System.out.println("Does 'city' exist? " + hasCity);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### Code Execution Output :
```
Name: Mahtab Alam
User Details:
name: Mahtab Alam
city: Bangalore
age: 31
Updated Age: 32
Fields: [city, name, age]
Values: [Mahtab Alam, 32, Bangalore]
After deleting 'city', fields: [name, age]
Does 'city' exist? false
```

# Redis List 
## RPUSH and LRANGE
```java
import redis.clients.jedis.Jedis;
public class RedisList {
    public static void main(String[] args) {
        try (Jedis jedis = new Jedis("localhost", 6379)) {
            String listKey = "myList";
            // Clear the list if it already exists
            jedis.del(listKey);

            // RPUSH to add elements at the end of the list(tail)
            jedis.rpush(listKey, "Value1");
            jedis.rpush(listKey, "Value2");
            jedis.rpush(listKey, "Value3");
            // LRANGE listKey 0 -1 retrieves all elements from start (0) to end (-1)
            System.out.println("All Elements in list: " + jedis.lrange(listKey, 0, -1));
            System.out.println("Last two elements : " + jedis.lrange(listKey, -2, -1));
        } catch (Exception e) {
            System.err.println("Error interacting with Redis: " + e.getMessage());
        }
    }
}
```
### Code Execution Output :
```
All Elements in list: [Value1, Value2, Value3]
Last two elements : [Value2, Value3]
```
