# Redis example with Jedis

Run a redis container using Docker (just make sure Docker Engine is running before executing the below command) 
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

## Example 1 : Redis storing and retrieving Strings
### set() , get(), setex()
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

## Example 2 : Redis Hash data type
### hset(), hget(), hgetall(), hkeys(), hvals(), hexists(), hdel()
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

# Example 3 : Redis List data type
## rpush(), lrange()
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
# Example 3a : Redis List data type
## lpush(), lrange()
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

# Example 3b : Redis List data type
## rpop(), lpop(), lindex(), lrange()
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

# Example 4 : Redis Set data type
## sadd() , smembers(), sismember(), srem()
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

# Example 5 : Redis Sorted Set data type
## zadd(), zrange(), zrangeWithScores(), zrangeByScore(), zrank(), zrem()
```java
import redis.clients.jedis.Jedis;
import redis.clients.jedis.resps.Tuple;
public class RedisSortedSetExample {

    public static void main(String[] args) {
        try (Jedis jedis = new Jedis("localhost", 6379)) {
            String sortedSetKey = "leaderboard";
            jedis.zadd(sortedSetKey, 100.0, "Player4");
            jedis.zadd(sortedSetKey, 20.0, "Player1");
            jedis.zadd(sortedSetKey, 0.0, "Player2");
            jedis.zadd(sortedSetKey, 200.0, "Player3");
            jedis.zadd(sortedSetKey, 150.0, "Player5");

            // Retrieve all elements by score (by default ascending order of score)
            System.out.println("Players sorted by rank (score):");
            for (String player : jedis.zrange(sortedSetKey, 0, -1)) {
                System.out.println(player);
            }

            // Retrieve all elements with scores
            System.out.println("\nPlayers with their scores:");
            for (Tuple entry : jedis.zrangeWithScores(sortedSetKey, 0, -1)) {
                System.out.println(entry.getElement() + " - Score: " + entry.getScore());
            }

            // Retrieve players with scores within a range
            System.out.println("\nPlayers with scores between 100 and 200:");
            for (String player : jedis.zrangeByScore(sortedSetKey, 100.0, 200.0)) {
                System.out.println(player);
            }

            // Get the rank of a specific player
            Long rank = jedis.zrank(sortedSetKey, "Player3");
            System.out.println("\nRank of Player3: " + rank);

            // Remove a player from the sorted set
            jedis.zrem(sortedSetKey, "Player1");
            System.out.println("\nRemoved Player1 from the sorted set");

            // Retrieve updated players
            System.out.println("Updated players in the sorted set:");
            for (String player : jedis.zrange(sortedSetKey, 0, -1)) {
                System.out.println(player);
            }

            // Clean up: delete the sorted set
            jedis.del(sortedSetKey);
            System.out.println("\nDeleted the sorted set");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### Code Execution Output :
```java
Players sorted by rank (score):
Player2
Player1
Player4
Player5
Player3

Players with their scores:
Player2 - Score: 0.0
Player1 - Score: 20.0
Player4 - Score: 100.0
Player5 - Score: 150.0
Player3 - Score: 200.0

Players with scores between 100 and 200:
Player4
Player5
Player3

Rank of Player3: 4

Removed Player1 from the sorted set
Updated players in the sorted set:
Player2
Player4
Player5
Player3

Deleted the sorted set
```

# Example 5a : Redis Sorted Set data type (sorting in descending order of score)
## zadd(), zrevrange(), zrevrangeWithScores(), zrevrangeByScore(), zrevrank(), zrem()
```java
import redis.clients.jedis.Jedis;
import redis.clients.jedis.resps.Tuple;
public class RedisSortedSetDescendingExample {

    public static void main(String[] args) {
        try (Jedis jedis = new Jedis("localhost", 6379)) {
            String sortedSetKey = "leaderboard";
            jedis.zadd(sortedSetKey, 100.0, "Player4");
            jedis.zadd(sortedSetKey, 20.0, "Player1");
            jedis.zadd(sortedSetKey, 0.0, "Player2");
            jedis.zadd(sortedSetKey, 200.0, "Player3");
            jedis.zadd(sortedSetKey, 150.0, "Player5");

            // Retrieve all elements by descending order of score
            System.out.println("Players sorted by score:");
            for (String player : jedis.zrevrange(sortedSetKey, 0, -1)) {
                System.out.println(player);
            }

            // Retrieve all elements with scores
            System.out.println("\nPlayers with their scores:");
            for (Tuple entry : jedis.zrevrangeWithScores(sortedSetKey, 0, -1)) {
                System.out.println(entry.getElement() + " - Score: " + entry.getScore());
            }

            // Retrieve players with scores within a range
            System.out.println("\nPlayers with scores between 200 and 100:");
            for (String player : jedis.zrevrangeByScore(sortedSetKey, 200.0, 100.0)) {
                System.out.println(player);
            }

            // Get the rank of a specific player
            Long rank = jedis.zrevrank(sortedSetKey, "Player3");
            System.out.println("\nRank of Player3: " + rank);

            // Remove a player from the sorted set
            jedis.zrem(sortedSetKey, "Player1");
            System.out.println("\nRemoved Player1 from the sorted set");

            // Retrieve updated players
            System.out.println("Updated players in the sorted set:");
            for (String player : jedis.zrevrange(sortedSetKey, 0, -1)) {
                System.out.println(player);
            }

            // Clean up: delete the sorted set
            jedis.del(sortedSetKey);
            System.out.println("\nDeleted the sorted set");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
### Code Execution Output :
```
Players sorted by score:
Player3
Player5
Player4
Player1
Player2

Players with their scores:
Player3 - Score: 200.0
Player5 - Score: 150.0
Player4 - Score: 100.0
Player1 - Score: 20.0
Player2 - Score: 0.0

Players with scores between 200 and 100:
Player3
Player5
Player4

Rank of Player3: 0

Removed Player1 from the sorted set
Updated players in the sorted set:
Player3
Player5
Player4
Player2

Deleted the sorted set
```

# Example 6 : Caching Java Object in Redis 

Redis stores data as strings, hashes, lists, sets, and other simple data types. To save a Java object, you must transform it into a byte stream or string that Redis can understand.

```java
// User.java
public class User {
    private String id;
    private String name;
    private int age;

    // Constructors
    public User() {
    }
    public User(String id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }
    // Getters and Setters
    public String getId() {
        return id;
    }
    public void setId(String id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    @Override
    public String toString() {
        return "name=" + name + ", id=" + id + ", age=" + age;
    }
}

// Test.java
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import redis.clients.jedis.Jedis;
public class Test {
    public static void main(String[] args) {
        // Jackson ObjectMapper for serialization and deserialization
        ObjectMapper objectMapper = new ObjectMapper();
        User user = new User("1", "Mahtab Alam", 31);

        try (Jedis jedis = new Jedis("localhost", 6379)) {
            // Serialize User object to JSON, and return as a String
            String userJsonAsString = objectMapper.writeValueAsString(user);
            System.out.println("Serialized User: " + userJsonAsString);

            String key = "user:1";
            jedis.set("user:1", userJsonAsString);
            // Retrieve JSON from Redis
            String retrievedJson = jedis.get(key);
            // Deserialize JSON String to User object
            if (retrievedJson != null) {
                User cachedUser = objectMapper.readValue(retrievedJson, User.class);
                System.out.println("Cached User: " + cachedUser);
            } else {
                System.out.println("Cache miss for key: " + key);
            }
        } catch (JsonProcessingException e) {
            System.err.println("Error in JSON processing: " + e.getMessage());
        } catch (Exception e) {
            System.err.println("Error connecting to Redis: " + e.getMessage());
        }
    }
}
```

### Code Execution Output :
```
Serialized User: {"id":"1","name":"Mahtab Alam","age":31}
Cached User: name=Mahtab Alam, id=1, age=31
```
