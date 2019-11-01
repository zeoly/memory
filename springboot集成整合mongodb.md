# springboot集成整合mongodb

在非关系型数据库的使用中，mongodb的出镜率现在也不小了，在最近做地图相关的项目中，关于海量轨迹数据的存储也选型了此数据库。在springboot中集成mongodb也非常简单，spring data模块提供了开箱即用的功能。

## 配置

首先向项目中引入依赖，修改`pom.xml`文件：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

并在配置文件`application.properties`中配置连接池相关参数：

```properties
spring.data.mongodb.uri=mongodb://username:password@localhost:27017/test_db
```

或者

```properties
spring.data.mongodb.username=username
spring.data.mongodb.password=password
spring.data.mongodb.database=test_db
spring.data.mongodb.host=localhost
spring.data.mongodb.prot=27017
```

通过以上两步，springboot应用到mongodb的连接就已经建立好了。

## 准备

首先定义一个对象，对应我们的存储内容

```java
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Data
@Builder
@Document(collection = "track_device")
public class TrackDevice {

    @Id
    private String id;
    
    private String location;
    
    private int weight;
}
```

> `@Document`注解表示此对象为文档，对应mongodb中的集合
>
> `@Id`注解声明此字段为主键

## 操作

随后的操作就非常简单了，只需要声明一个接口来继承`MongoRepository`，即可开始使用spring-data中的各种便利：

```java
@Repository
public interface TrackDeviceRepository extends MongoRepository<TrackDevice, String> {

    List<TrackDevice> findByLocation(String location);
    
    @Query(value = "{'weight': {'$gt':?0, '$lt':?1}}")
    List<TrackDevice> findByWeightRange(int start, int end);

}
```

> 如果遇到一些复杂操作，可以注入`MongoTemplate`来进行一些自定义操作。