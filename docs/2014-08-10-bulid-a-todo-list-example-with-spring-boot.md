# Bulid a Todo List Example  with Spring Boot

[Spring Boot](http://projects.spring.io/spring-boot/)利用JavaConfig的配置模式以及“约定优于配置”的理念，能够极大的简化基于Spring MVC的Web应用和REST服务开发。使用Spring Boot能够很容易的创建独立的Spring应用程序，它直接嵌入Tomcat或者Jetty，不需要部署WAR文件，不需要XML文件，能够简化配置，极大的方便用户来快速创建一个应用程序。下面我将介绍如何使用Spring Boot创建一个简单的RESTful API。这个例子是一个简单的todo list，可以实现对todo list的增删查改。例如：

#### 运行程序：

```
spring-boot:run
```
#### 增加一条todo：

```
curl -X "POST" -v localhost:8080/todos -H "Content-Type: application/json" -d '{"description":"This is a todo."}'
```

#### 查询所有的todo：

```
curl -X "GET" -v localhost:8080/todos
```
#### 删除一条todo：

```
curl -X "DELETE" -v localhost:8080/todos/1
```

#### 修改一条todo：

```
curl -X "PUT" -v localhost:8080/todos/2 -H "Content-Type: application/json" -d '{"description":"Modified Todo"}'
```

在这个例子中，我们使用Spring Date JPA处理数据，使用mysql存储数据：

```
[
  {
    "id": 1,
    "description": "A Todo"
  },
  {
    "id": 2,
    "description": "Study Spring Boot. "
  }
]
```

## 创建文件结构

可以使用[Spring Initializr](http://start.spring.io/)来帮助你快速创建出本工程的文件结构和POM.xml文件，也可以手动创建。创建完成后，本例子的文件结构如下：

```
└──src
    └──main
	   └──java
	       └──todos
	       └──resources
```

## pom.xml
工程创建成功之后，需要添加pom.xml文件（如果使用Spring Initializr会自动生成pom.xml文件，只需要做一些修改即可），本例的pom.xml文件内容如下：

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.1.4.RELEASE</version>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.6</version>
        </dependency>
    </dependencies>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <start-class>todos.Application</start-class>
    </properties>
    <build>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.3.2</version>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    <repositories>
        <repository>
            <id>spring-releases</id>
            <name>Spring Releases</name>
            <url>http://repo.spring.io/libs-release</url>
        </repository>
        <repository>
            <id>org.jboss.repository.releases</id>
            <name>JBoss Maven Release Repository</name>
            <url>https://repository.jboss.org/nexus/content/repositories/releases</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-releases</id>
            <name>Spring Releases</name>
            <url>http://repo.spring.io/libs-release</url>
        </pluginRepository>
    </pluginRepositories>
```

在上面的配置中，需要将工程的parent被设置为spring-boot-starter-parent，并添加对spring-boot-starter-web，spring-boot-starter-data-jpa，mysql-connector-java的依赖。spring-boot-starter-parent会继承一些默认的依赖，这样的设置之后就无需添加一堆相应的依赖，能够将依赖配置最小化；spring-boot-starter-web提供了对web的支持；spring-boot-starter-data-jpa提供了对数据的支持；mysql-connector-java提供了和后台mysql数据库连接的支持。在构建中要声明使用spring-boot-maven-plugin这个插件，它会对Maven打包形成的JAR进行二次修改，最终产生符合我们要求的内容结构，并且可以直接使用mvn spring-boot: run运行应用程序。

## 实体

```java
package todos;

import javax.persistence.*;

@Entity
@Table(uniqueConstraints = { @UniqueConstraint(columnNames = { "id" }) })
public class Todo {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;
    private String description;

    public void setDescription(String description) {
        this.description = description;
    }

    public long getId() {
        return id;
    }

    public String getDescription() {
        return description;
    }
}
```

使用@Entity表明该实体为JPA实体，它包含id和description两个属性，其中id为自动生成，其结构对应于数据库中Todo表。创建完成后，我们就可以利用此实体方便的进行操作。

## Controller

拥有Todo这个实体之后，我们需要一个Controller来响应实体的请求：

```java
package todos;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import static org.springframework.web.bind.annotation.RequestMethod.*;

@RestController
@RequestMapping("/todos")
public class TodoController {
    @Autowired
    TodoService todoService;

    @RequestMapping(method = GET)
    public Iterable<Todo> showTodoList() {
        return todoService.findAllList();
    }

    @RequestMapping(value = "/{id}", method = GET)
    public Todo showATodo(@PathVariable(value = "id") Long id) {
        return todoService.findById(id);
    }

    @RequestMapping(method = POST)
    public Todo create(@RequestBody Todo newTodo) {
        return todoService.createNewTodo(newTodo);
    }

    @RequestMapping(value = "/{id}" ,method = DELETE)
    public void delete(@PathVariable("id") Long id){
        todoService.delete(id);
    }

    @RequestMapping(value = "/{id}" , method = PUT)
    public Todo update(@RequestBody Todo updateTodo, @PathVariable("id") Long id){
        return todoService.updateTodo(updateTodo, id);
    }
}
```

其中@RestController是Spring 4中的annotation，它集合了@Controller和@ResponseBody的作用，能够标注一个类是Controller，并且是该类的方法返回一个domain对象。@RequestMapping用于确保该类中方法map到对应的路径。@PathVariable会将url中的参数解析到对应的方法参数上，使用它时需要在@RequestMapping()指定匹配模式 ，例如：`@RequestMapping(value = "/{id}" ,method = DELETE)`，当使用`/1`时，就可以把“1”解析到方法参数id上。

传统Spring MVC的Controller与RESTful Web服务器的Controller最大的区别在于建立HTTP响应的方式。这个例子中的Controller只使用Todo对象进行传入和返回参数，Todo对象以JSON的形式传输。

## Service

有了Controller之后，我们需要一个Service来处理Controller与Repository之间的交互。代码如下：

``` java
package todos;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Service;

@Service
public class TodoService {
    @Qualifier("todoRepository")
    @Autowired
    TodoRepository todoRepository;

    public Iterable<Todo> findAllList(){
        return todoRepository.findAll();
    }
    public Todo createNewTodo(Todo todo){
        return todoRepository.save(todo);
    }
    public Todo findById(Long id){
        return todoRepository.findOne(id);
    }
    public void delete(Long id){
        todoRepository.delete(id);
    }
    public Todo updateTodo(Todo todo, Long id){
        if(id != todo.getId())
            todoRepository.delete(id);
        return todoRepository.save(todo);
    }
}
```

可以看到，该Service中Autowired了TodoRepository，并利用其实现了对Todo的增删查改。

## Repository

本例中使用Spring Data JPA进行数据存储，它关注在关系型数据库使用JPA存储数据，在运行时从一个repository接口中自动创建repository实例。

``` java
package todos;

import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Component;

@Component
public interface TodoRepository extends CrudRepository<Todo, Long> {
}
```

本例子中的`TodoRepository`继承了`CrudRepository`接口，通过泛型指定了实体类型是Todo和Long。`CrudRepository`包括多个持久化方法，包括保存、删除、查找和更新Todo实体。`TodoRepository`继承了这些方法，并且可以在此基础上修改或者增加一些新的方法。

## 配置文件

Spring Boot不需要xml配置文件，只需要在resources目录下添加一个application.yml文件来进行相应的配置，例如使用：

``` yml
spring:
  profiles: development

  datasource:
    driverClassName: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/todos
    username: root
    password:

  jpa:
     show-sql: true
     hibernate:
       ddl-auto: update
```

设置本例中使用jpa和mysql.

## 启动应用

``` java
package todos;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;

@EnableAutoConfiguration
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

Application类的main方法中使用了SpringApplication帮助类，并以Application这个类作为配置来启动Spring的应用上下文。`@EnableAutoConfiguration`用于开启自动配置，使用它可以使我们刚刚设置的配置文件自动生效。

