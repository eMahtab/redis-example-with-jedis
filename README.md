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
## RPUSH
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
## LPUSH
```java
import redis.clients.jedis.Jedis;
public class RedisList {
    public static void main(String[] args) {
        try (Jedis jedis = new Jedis("localhost", 6379)) {
            String listKey = "myList";
            // Clear the list if it already exists
            jedis.del(listKey);

            // LPUSH to add elements at the head of the list
            jedis.lpush(listKey, "Value1");
            jedis.lpush(listKey, "Value2");
            jedis.lpush(listKey, "Value3");
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
All Elements in list: [Value3, Value2, Value1]
Last two elements : [Value2, Value1]
```

## Redis List : rpop(), lpop(), lindex(), lrange()
```java
import redis.clients.jedis.Jedis;
public class ListOperations {
    public static void main(String[] args) {
        try (Jedis jedis = new Jedis("localhost", 6379)) {
            String listKey = "myList";
            // Clear the list if it already exists
            jedis.del(listKey);

            // RPUSH to add elements at tail, LPUSH for adding at the head
            jedis.rpush(listKey, "Value1");
            jedis.rpush(listKey, "Value2");
            jedis.rpush(listKey, "Value3");
            jedis.lpush(listKey, "Value0");

            System.out.println("First element at the head :" + jedis.lindex(listKey, 0));
            System.out.println("First 5 elements from the head:" + jedis.lrange(listKey, 0, 4));
            System.out.println("Last 3 elements from the end:" + jedis.lrange(listKey, -3, -1));
            System.out.println("Pop last element from end of the list :" + jedis.rpop(listKey));
            System.out.println("Pop first element from the head of the list :" + jedis.lpop(listKey));
        } catch (Exception e) {
            System.err.println("Error interacting with Redis: " + e.getMessage());
        }
    }
}
```

### Code Execution Output :
```
First element at the head :Value0
First 5 elements from the head:[Value0, Value1, Value2, Value3]
Last 3 elements from the end:[Value1, Value2, Value3]
Pop last element from end of the list :Value3
Pop first element from the head of the list :Value0
```

# Redis Set :
```java
import redis.clients.jedis.Jedis;
public class RedisSetExample {
    public static void main(String[] args) {
        try (Jedis jedis = new Jedis("localhost", 6379)) {
            String setKey = "mySet";
            jedis.sadd(setKey, "element1", "element2", "element3");
            jedis.sadd(setKey, "element4");
            System.out.println("Elements in the set:");
            for (String element : jedis.smembers(setKey)) {
                System.out.print(element + " ");
            }
            System.out.println();

            String elementToCheck = "element2";
            boolean exists = jedis.sismember(setKey, elementToCheck);
            System.out.println(elementToCheck + " exists in the set? " + exists);

            // Remove an element from the set
            jedis.srem(setKey, "element1");
            System.out.println("Removed 'element1' from the set");

            // Retrieve and display the updated set
            System.out.println("Updated elements in the set:");
            for (String element : jedis.smembers(setKey)) {
                System.out.println(element);
            }
            // Clean up: delete the set
            jedis.del(setKey);
            System.out.println("Deleted the set");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### Code Execution Output :
```
Elements in the set:
element1 element2 element3 element4 
element2 exists in the set? true
Removed 'element1' from the set
Updated elements in the set:
element2
element3
element4
Deleted the set
```
