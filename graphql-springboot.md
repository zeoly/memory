[toc]

# Spring Boot与GraphQL - 篇三 集成开发

在了解了GraphQL相关概念，以及使用的方法后，就该进入到实操阶段了。使用常用的springboot来搭建服务，并提供GraphQL查询。

## 配置

在Spring Boot中集成GraphQL非常简单，只需引入两个包

*pom.xml*

```xml
<dependency>
	<groupId>com.graphql-java</groupId>
	<artifactId>graphql-spring-boot-starter</artifactId>
	<version>4.0.0</version>
</dependency>
<dependency>
	<groupId>com.graphql-java</groupId>
	<artifactId>graphql-java-tools</artifactId>
	<version>4.3.0</version>
<dependency>
```

## 定义Schema

推荐在`resources`目录下建立`graphql`文件夹，在其中建立`root.graphqls`和`schema.graphqls`两个文件。一般会在`root.graphqls`文件中放`Query`或者`Mutation`的接口定义，在`schema.graphqls`文件中定义`type`等数据对象。

*root.graphqls*

```
type Query {
    getUserById(id: Int!): User
    listUser: [User]
}
 
type Mutation {
    saveUser(user: addUserInput!): Boolean
    deleteUser(id: Int!): Boolean
    updateUser(user: updateUserInput!): Boolean
}
```



*schema.graphqls*

```
type User {
    id: Int!
    username: String!
    password: String!
    age: Int
}
 
input addUserInput {
    id: Int!
    username: String!
    password: String!
    age: Int
}
 
input updateUserInput {
    id: Int!
    username: String!
    password: String!
    age: Int
}
```

> 一般都会有多个数据对象，建议schema的定义可以在使用对象来分文件，例如`user.graphqls`

## 查询与变更的实现

GraphQL只是提供了数据获取以及数据操作的一个新的方式入口，所以原有代码逻辑与结构都不会有任何变更，只是需要实现` GraphQLQueryResolver `和` GraphQLMutationResolver `接口，来实现对应`Query`和`Mutation`中定义的入口，在这里我们直接使用DAO层来对接。

*UserQueryResovler.java*

```java
@Component
public class UserQueryResovler implements GraphQLQueryResolver {
    
    @Autowired
    private UserRepository userRepository;
    
    public User getUserById(Integer id) {
        return userRepository.get(id);
    }
    
    public List<User> listUser() {
        return userRepository.findAll();
    }
}
```

*UserMutationResolver.java*

```java
@Component
public class UserMutationResolver implements GraphQLMutationResolver {
    
    @Autowired
    private UserRepository userRepository.save(user);;
    
    public boolean saveUser(User user) {
        return userRepository.save(user);
    }
    
    public boolean deleteUser(Integer id) {
        return userRepository.deleteById(id);
    }
    
    public boolean updateUser(User user) {
        return userRepository.update(user);
    }
}
```

> 需要注意返回类型、方法名、参数类型都需要和schema中的定义一致。

## 启动测试

在如上三个步骤后，我们已经完成了相关的改造，可以启动进行测试了。直接启动应用，通过`/graphql`即可进行访问，例如`http://localhost:8080/graphql`。

### 使用postman测试

可以使用`get`或者`post`请求的方式，如果是`post`方式，请求体数据格式为`json`。有三个参数：`query`、`variavles`和`operationName`，其中`query`是必传参数。例如：

*get请求*

```
http://localhost:8080/graphql?query={viewer{name}}
```

*post请求体*

```json
{
  "query": "{viewer{name}}",
  "operationName": "",
  "variables": { "name": "value", ... }
}
```



### 可视化页面测试

还有一个更直接的测试方式是使用可视化页面进行，需要多引入一个包：

*pom.xml*

```xml
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphiql-spring-boot-starter</artifactId>
    <version>4.0.0</version>
</dependency>
```

然后启动应用即可，通过类似`http://localhost:8080/graphiql`进行访问：