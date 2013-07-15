
Getting Started: Accessing Data with Mongo
==========================================

What you'll build
-----------------

This guide walks you through the process of building an application Spring Data Mongo to store and retrieve data in mongo's document-based database.

What you'll need
----------------

 - About 15 minutes
 - A favorite text editor or IDE
 - [JDK 6][jdk] or later
 - [Maven 3.0][mvn] or later

[jdk]: http://www.oracle.com/technetwork/java/javase/downloads/index.html
[mvn]: http://maven.apache.org/download.cgi

How to complete this guide
--------------------------

Like all Spring's [Getting Started guides](/getting-started), you can start from scratch and complete each step, or you can bypass basic setup steps that are already familiar to you. Either way, you end up with working code.

To **start from scratch**, move on to [Set up the project](#scratch).

To **skip the basics**, do the following:

 - [Download][zip] and unzip the source repository for this guide, or clone it using [git](/understanding/git):
`git clone https://github.com/springframework-meta/gs-accessing-data-mongo.git`
 - cd into `gs-accessing-data-mongo/initial`
 - Jump ahead to [Install and launch Mongo](#initial).

**When you're finished**, you can check your results against the code in `gs-accessing-data-mongo/complete`.
[zip]: https://github.com/springframework-meta/gs-accessing-data-mongo/archive/master.zip


<a name="scratch"></a>
Set up the project
------------------

First you set up a basic build script. You can use any build system you like when building apps with Spring, but the code you need to work with [Maven](https://maven.apache.org) and [Gradle](http://gradle.org) is included here. If you're not familiar with either, refer to [Building Java Projects with Maven](../gs-maven/README.md) or [Building Java Projects with Gradle](../gs-gradle/README.md).

### Create the directory structure

In a project directory of your choosing, create the following subdirectory structure; for example, with `mkdir -p src/main/java/hello` on *nix systems:

    └── src
        └── main
            └── java
                └── hello

### Create a Maven POM

`pom.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-acessing-data-mongo-complete</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.bootstrap</groupId>
        <artifactId>spring-bootstrap-starters</artifactId>
        <version>0.5.0.BUILD-SNAPSHOT</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.bootstrap</groupId>
            <artifactId>spring-bootstrap-web-starter</artifactId>
        </dependency>
    	<dependency>
	    	<groupId>org.springframework.data</groupId>
	    	<artifactId>spring-data-mongodb</artifactId>
	    	<version>1.2.1.RELEASE</version>
    	</dependency>
    </dependencies>

    <properties>
        <start-class>hello.Application</start-class>
    </properties>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    
    <!-- TODO: remove once bootstrap goes GA -->
    <repositories>
        <repository>
            <id>spring-snapshots</id>
            <name>Spring Snapshots</name>
            <url>http://repo.springsource.org/snapshot</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
        <repository>
            <id>neo4j</id>
            <name>Neo4j</name>
            <url>http://m2.neo4j.org/</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-snapshots</id>
            <name>Spring Snapshots</name>
            <url>http://repo.springsource.org/snapshot</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </pluginRepository>
    </pluginRepositories>
</project>
```


<a name="initial"></a>
Install and launch Mongo
------------------------
With your project setup, the next step is to install and launch the Mongo database.

If you are using a Mac with homebrew, this is as simple as:

    $ brew install mongodb
    
With MacPorts:

    $ port install mongodb
    
For other systems with package management, like Redhat, Ubuntu, Debian, CentOS, Windows, and others, more instructions are included at http://docs.mongodb.org/manual/installation/.

Once you have installed it, you can launch it immediately in a console window:

    $ mongod
    
You probably won't see much more than this:

```sh
all output going to: /usr/local/var/log/mongodb/mongo.log
```
    
This starts up a server process. 


Define a simple entity
------------------------
Mongo is a NoSQL document store. In this example, you store `Customer` objects.

`src/main/java/hello/Customer.java`
```java
package hello;

import org.springframework.data.annotation.Id;


public class Customer {

	@Id
    private String id;
	
    private String firstName;
    private String lastName;

    public Customer() {}

    public Customer(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    @Override
    public String toString() {
        return String.format(
                "Customer[id=%s, firstName='%s', lastName='%s']",
                id, firstName, lastName);
    }

}

```

Here you have a `Customer` class with three attributes, the `id, the `firstName` and the `lastName`. The `id` is mostly for internal use by Mongo. You also have a single constructor to populate the entities when creating a new instance.

> Note: In this guide, the typical getters and setters have been left out for brevity.

`id` fits the standard name for a Mongo id so it doesn't require any special annotation to tag it for Spring Data Mongo.

The other two properties, `firstName` and `lastName` are left unannotated. It is assumed that they'll be mapped to columns that share the same name as the properties themselves.

The convenient `toString()` method will print out the details about a customer.

> **Note:** Mongo stores data in collections. Spring Data Mongo will map the class `Customer` into a collection called **customer**. If you want to change the name of the collection, you can use Spring Data Mongo's [`@Document`](http://static.springsource.org/spring-data/data-mongodb/docs/current/api/org/springframework/data/mongodb/core/mapping/Document.html) annotation on the class.

Create simple queries
----------------------------
Spring Data Mongo focuses on storing data in Mongo. It also inherits powerful functionality from the Spring Data Commons project, such as the ability to derive queries. Essentially, you don't have to learn the query language of Mongo; you can simply write a handful of methods and the queries are written for you.

To see how this works, create a repository interface that queries `Customer` documents.

`src/main/java/hello/CustomerRepository.java`
```java
package hello;

import java.util.List;

import org.springframework.data.mongodb.repository.MongoRepository;

public interface CustomerRepository extends MongoRepository<Customer, String> {
	
	public Customer findByFirstName(String firstName);
	public List<Customer> findByLastName(String lastName);
	
}
```
    
`CustomerRepository` extends the `MongoRepository` interface and plugs in the type of values and id it works with: `Customer` and `String`. Out-of-the-box, this interface comes with many operations, including standard CRUD operations (change-replace-update-delete).

You can define other queries as needed by simply declaring their method signature. In this case, you add `findByFirstName`, which essentially seeks documents of type `Customer` and finds the one that matches on `firstName`.

You also have:
- `findByLastName` to find a list of people by last name

In a typical Java application, you would write a class that implements `CustomerRepository` and craft the queries yourself. What makes Spring Data Mongo so powerful is the fact that you don't have to create this implementation. Spring Data Mongo creates it on the fly when you run the application.

Let's wire this up and see what it looks like!

Create an application class
---------------------------
Here you create an Application class with all the components.

`src/main/java/hello/Application.java`
```java
package hello;

import java.net.UnknownHostException;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.repository.config.EnableMongoRepositories;

import com.mongodb.Mongo;


@Configuration
@EnableMongoRepositories
public class Application {
	
	@Autowired
	CustomerRepository customerRepository;
	
	@Bean
	Mongo mongo() throws UnknownHostException {
		return new Mongo("localhost");
	}
	
	@Bean
	MongoTemplate mongoTemplate(Mongo mongo) {
		return new MongoTemplate(mongo, "gs-accessing-data-mongo");
	}

    public static void main(String[] args) {
        AbstractApplicationContext context = new AnnotationConfigApplicationContext(Application.class);
        CustomerRepository repository = context.getBean(CustomerRepository.class);
        
        repository.deleteAll();
        
        // save a couple of customers
        repository.save(new Customer("Alice", "Smith"));
        repository.save(new Customer("Bob", "Smith"));
        
        // fetch all customers
        System.out.println("Customers found with findAll():");
        System.out.println("-------------------------------");
        for (Customer customer : repository.findAll()) {
            System.out.println(customer);
        }
        System.out.println();
        
        // fetch an individual customer
        System.out.println("Customer found with findByFirstName('Alice'):");
        System.out.println("--------------------------------");
        System.out.println(repository.findByFirstName("Alice"));

        System.out.println("Customers found with findByLastName('Smith'):");
        System.out.println("--------------------------------");
        for (Customer customer : repository.findByLastName("Smith")) {
        	System.out.println(customer);
        }

        context.close();
    }
    
}
```

In the configuration, you need to add the `@EnableMongoRepositories` annotation. This tells Spring Data Mongo that it should seek out any interface that extends `org.springframework.data.repository.Repository` and to automatically generate an implementation. By extending `MongoRepository`, your `CustomerRepository` interface transitively extends `Repository`. Therefore, Spring Data Mongo will find it and create an implementation for you.

* The `Mongo` connection links the application to your Mongo database server
* The `MongoTemplate` is needed by Spring Data Mongo to execute the queries behind your `find*` methods. You can use the template yourself for more complex queries, but this guide won't go into that.
* Finally, you autowire an instance of `CustomerRepository`. Spring Data Mongo dynamically creates a proxy and injects it there.

Finally, `Application` includes a `main()` method that puts the `CustomerRepository` through a few tests. First, it fetches the `CustomerRepository` from the Spring application context. Then it saves a handful of `Customer` objects, demonstrating the `save()` method and setting up some data to work with. Next, it calls `findAll()` to fetch all `Customer` objects from the database. Then it calls `findByFirstName()` to fetch a single `Customer` by her first name. Finally, it calls `findByLastName()` to find all customers whose last name is "Smith".

Now that your `Application` class is ready, you simply instruct the build system to create a single, executable jar containing everything. This makes it easy to ship, version, and deploy the service as an application throughout the development lifecycle, across different environments, and so forth.

Add the following configuration to your existing Maven POM:

`pom.xml`
```xml
    <properties>
        <start-class>hello.Application</start-class>
    </properties>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

The `start-class` property tells Maven to create a `META-INF/MANIFEST.MF` file with a `Main-Class: hello.Application` entry. This entry enables you to run the jar with `java -jar`.

The [Maven Shade plugin][maven-shade-plugin] extracts classes from all jars on the classpath and builds a single "über-jar", which makes it more convenient to execute and transport your service.

Now run the following to produce a single executable JAR file containing all necessary dependency classes and resources:

    mvn package

[maven-shade-plugin]: https://maven.apache.org/plugins/maven-shade-plugin
    
Run the application
-------------------
Run your application with `java -jar` at the command line:

    java -jar target/gs-accessing-data-mongo-0.1.0.jar


    
You should see something like this (with other stuff like queries as well):
```
Customers found with findAll():
-------------------------------
Customer[id=51df1b0a3004cb49c50210f8, firstName='Alice', lastName='Smith']
Customer[id=51df1b0a3004cb49c50210f9, firstName='Bob', lastName='Smith']

Customer found with findByFirstName('Alice'):
--------------------------------
Customer[id=51df1b0a3004cb49c50210f8, firstName='Alice', lastName='Smith']
Customers found with findByLastName('Smith'):
--------------------------------
Customer[id=51df1b0a3004cb49c50210f8, firstName='Alice', lastName='Smith']
Customer[id=51df1b0a3004cb49c50210f9, firstName='Bob', lastName='Smith']
```

Summary
-------
Congratulations! You set up a Mongo server, written a simple application that uses Spring Data Mongo to save some objects to a database, and to fetch them; all without writing a concrete repository implementation.