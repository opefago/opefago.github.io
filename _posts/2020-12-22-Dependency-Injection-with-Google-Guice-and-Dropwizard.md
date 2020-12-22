---
layout : post
title: 'Dependency Injection with Google Guice and Dropwizard'
date: '2020-12-22 07:18:33'
tags: dropwizard, dependency Injection, guice, guicey
subclass: 'post tag-dropwizard, dependency Injection, guice, guicey'
categories: ''
cover: ''

---

Dropwizard is a lightweight framework for building fast and reliable RESTful services in Java, it is an opinionated framework that comes configured with libraries such as Jetty, hibernate, jersey, Jackson, metrics etc. 

In my opinion, there are two main challenges I have experienced when it comes to development using dropwizard, first of, there are not so much help and materials when its comes to learning the core of the framework and secondly, development can get repetitive and boring, for those who are coming from Spring boot, one of the first things you will miss is Dependency Injection, fortunately, Google has a framework just for this, it’s called [Google Guice](https://github.com/google/guice). Google Guice is a library that aids Java development experience without the need to both about creating factories or use of the new Keyword. Integrating Guice with dropwizard helps to reduce some of the bore that comes with using the dropwizard framework.

The first time I tried to integrate Guice with dropwizard, I struggled a little bit. I decided to document my experience so that others don’t have to suffer as I did ;)

**Getting Started With Guice**

SOLID principle highlights that while designing classes, they should have Single Responsibility as well as dependency inversion. Dependency Injection as a programming technique helps us achieve this by decoupling Object Creation from its usage as well as abstracting how the low level details are implemented, this helps to write clean code and also aids testing. For example, You have a class that turns on the light bulb called Bulb, and another that manages the smart home called SmartHome, SmartHome needs to turn on the light as well as handle other electrical interfaces in the house, SmartHome should not know anything about turning the lighting off and on, that is not its responsibility, it should only know how to talk to classes/interfaces that directly know how to trigger this actions. 

NB: In reality, there should be an Interface that all other electrical appliance Class implements, it makes it easy for the SmartHome Object to talk to call a single method when interacting with its dependencies, but for this example, SmartHome will be interacting directly with the Bulb class.

The logic for the turning on and off the light Bulb will be implemented in the Bulb class and the SmartHome class takes in this object in its constructor when it needs to be used, a Bulb object is created and passed into a SmartHome object. This might be okay when you are interacting with 1 or 2 objects but in production-ready apps, you might have hundreds and thousands of objects that need interaction, this will make your project cumbersome and it can get really hard to manage. This is where Dependency Injection libraries shines, you make use of annotations like `@Inject` when creating the classes and the library helps you to wire them automatically.

Guice is a framework that helps you to easily manage your dependencies and 

**Guice and Dropwizard**

For this exercise, we will be building a simple service that stores values in a key-value database using Redis, this gives us a full grasp of how to wire configurations to services that require them as well as how to create dependencies for services. By the end of the articles, you would be very comfortable using Guice and dropwizard.

**Prerequisite**

To nicely integrate Guice into dropwizard, we will be making using of a very decent integration library called [dropwizard guice](https://github.com/xvik/dropwizard-guicey). This helps us to automatically integrate various moving part of the dropwizard framework using classpath scan. This would be familiar to you if you are coming from a Spring boot background. 

We will be using [reddisson](https://github.com/redisson/redisson) A very good Redis client that provides high-performance async read and writes and provides TTL queues, retry queues and a host of other kinds of use cases right out of the box.

We will be developing our dropwizard application using Java 8 as well as Gradle, if you prefer Maven, you can also follow the example without problems and IntelliJ as the IDE for development.

**Gradle Dependencies**

Create a new project and In addition to the dropwizard-core add the following dependencies to our gradle file:

```groovy
implementation 'ru.vyarus:dropwizard-guicey:5.2.0'
implementation 'org.redisson:redisson:3.14.0'
```

And also, I used lombok to ease development, hence, the following dependencies should also be added;

```groovy
compileOnly 'org.projectlombok:lombok:1.18.16'
annotationProcessor 'org.projectlombok:lombok:1.18.16'
testCompileOnly 'org.projectlombok:lombok:1.18.16'
testAnnotationProcessor 'org.projectlombok:lombok:1.18.16'

```

**Configuration**

When working with Dropwizard, one of the first things you need to setup is the configuration file, create a file named app.yml in the project root directory and add the following to it:

```yaml
server: 
	applicationConnectors:  
		- type: http   
			port: 9000 
	adminConnectors:  
		- type: http   
			port: 9001
```


Next create a class called `AppConfiguration.java` where we can access the configuration parameters with the following:

```java
package com.opefago.di.config;

import io.dropwizard.Configuration;
import lombok.Getter;
import javax.validation.constraints.NotNull;

@NotNull
@Getterpublic 
class AppConfiguration extends Configuration {}
```



**Application**

Create an `App.java` class and add this base application to setup a running dropwizard application:

```java
package com.opefago.di;
import com.opefago.di.config.AppConfiguration;
import io.dropwizard.Application;
import io.dropwizard.setup.Environment;
public class App extends Application<AppConfiguration> { 	
  public static void main(String[] args) throws Exception {
    new App().run(args);  
  }
  @Override
  public void run(AppConfiguration configuration, Environment environment) throws Exception {
  }
}
```

**Running the Application**

Before we can run the application, we have to setup the gradle file to have the right plugins for bootstrapping a dropwizard project, this applies if you are using gradle, of you are using maven, the setup is slightly different, you can refer to the tutorial [here](https://www.baeldung.com/java-dropwizard):

```groovy
import java.time.ZoneId
import java.time.ZonedDateTime
import java.time.format.DateTimeFormatter
import java.time.ZoneId
import java.time.ZonedDateTime
import java.time.format.DateTimeFormatter

plugins {  
  id 'java'  
  id 'com.github.johnrengelman.shadow' version '5.2.0'
}

group 'com.opefago.di'
version '1.0'
apply plugin: 'java'
apply plugin: 'application'
mainClassName = 'com.opefago.di.App'
sourceCompatibility = 1.8
targetCompatibility = 1.8
repositories {  
  mavenCentral()
}
def versions = [    
  dropwizard : "1.3.14",
  guice: "5.2.0",
  lombok: "1.18.16"
]
dependencies {  
  testCompile group: 'junit', name: 'junit', version: '4.12'
  implementation "ru.vyarus:dropwizard-guicey:$versions.guice"
  implementation "org.redisson:redisson:3.14.0"
  compileOnly "org.projectlombok:lombok:$versions.lombok"
  annotationProcessor "org.projectlombok:lombok:$versions.lombok"
  testCompileOnly "org.projectlombok:lombok:$versions.lombok"
  testAnnotationProcessor "org.projectlombok:lombok:$versions.lombok"
  implementation "io.dropwizard:dropwizard-core:$versions.dropwizard"
}
shadowJar {  
  mergeServiceFiles()  
  exclude 'META-INF/*.DSA', 'META-INF/*.RSA', 'META-INF/*.SF'  
  manifest {    
    attributes 'Implementation-Title': rootProject.name    
    attributes 'Implementation-Version': rootProject.version    
    attributes 'Implementation-Vendor-Id': rootProject.group    
    attributes 'Build-Time': ZonedDateTime.*now*(ZoneId.*of*("UTC")).format(DateTimeFormatter.*ISO_ZONED_DATE_TIME*)    
    attributes 'Built-By': InetAddress.*localHost*.hostName    
    attributes 'Created-By': 'Gradle ' + gradle.gradleVersion    
    attributes 'Main-Class': "com.opefago.di.App"  
  }
}
```

With this, we should have a minimal running dropwizard app that can run on port 9000, let's take it for a spin before we progress, run the following command on the terminal

```sh
 ./gradlew build
```

This should generate a jar file that is post fixed what ‘-all’ in the build/libs folder. Now run the app with the following command:

```sh
java -jar build/libs/dropwizard-guice-di-1.0-all.jar server app.yml
```

If all things go well, you should have a running application that does nothing interesting yet. Now Let’s integrate google guice into it and afterwards set up a service and a resource to see the fill power of google guice in display, we will conclude by also adding healthcheck as well.

**Adding Google Guice**

In the `App.java` class, add the the initialise method as shown below:

```java
@Override
public void initialize(Bootstrap<AppConfiguration> bootstrap) {
  bootstrap.addBundle(
    GuiceBundle.builder().enableAutoConfig(getClass().getPackage().getName()).build()
  );
}
```

This is going to add a dropwizard bundle that scans your project path for all Managed, healthcheck, etc as well as their dependencies, as soon as this is done, all you need is to write your code and it automatically picks them, this makes your project simple enough and leave your main class decluttered from too many boilerplate.

**Services**

This is practically all you need to do to get set up with guice on dropwizard, but in other to get a feel of the power that guice brings into dropwizard, let's proceed to create a service that communicates with redis. Create a class called `RedisService`, I prefer my services to be implementation of Managed interface provided by dropwizard, so that they can be plugged into the dropwizard lifecycle, you can used the start and stop method to ensure that dependencies are properly managed by the dropwizard lifecycle and more importantly, so that they can be automatically scanned and picked up by Guicey without any extra work.Our code should be looking like so:

```java
package com.opefago.di.services;
import io.dropwizard.lifecycle.Managed;
public class RedisService implements Managed {
  @Override  
  public void start() throws Exception {
  }
  @Override  
  public void stop() throws Exception {
  }
}
```

As I mentioned earlier, I'll be using redisson client for connecting to redis. There are other redis clients available out there, I just like that redission provides all types of queue types out of the box. Before we get started, add the following configuration to app.yaml file

```yaml
redis: 
	uri: ${REDIS_URI:-localhost} 
	port: ${REDIS_PORT:-6379} 
	idleConnectionTimeout: 10000 
	connectTimeout: 10000 
	timeout: 3000 
	retryAttempts: 3 
	retryInterval: 1500
```

Notice how the uri and port fields are specified, this indicates that they need to be substituted by environment variables with defaults if the environment variables are not found. Environment variable swapping does not come out of the box in dropwizard, if you run this, the variables will be presented as specified, you have to make dropwizard aware of this behavior. To do that, add the following code to the initialize method of our main class:

```java
bootstrap.setConfigurationSourceProvider(    
  new SubstitutingSourceProvider(bootstrap.getConfigurationSourceProvider(),        
                                 new EnvironmentVariableSubstitutor(false)    
                                )
);
```

Next, You have to create a `RedisCongiration` class that maps the files in the app.yml to class parameters and methods, create a class like so:

```java
import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.Getter;
import javax.validation.constraints.NotNull;

@NotNull
@Getter
public class RedisConfiguration {  
  @JsonProperty("uri") private String uri; 
  @JsonProperty("port") private String port;   
  @JsonProperty("idleConnectionTimeout") private int idleConnectionTimeout;
  @JsonProperty("connectTimeout") private int connectTimeout;  
  @JsonProperty("timeout") private int timeout;  
  @JsonProperty("retryAttempts") private int retryAttempts;  
  @JsonProperty("retryInterval") private int retryInterval;
}
```

 And then update your AppConfiguration class like so:

```java
public class AppConfiguration extends Configuration {  
  @JsonProperty("redis") private RedisConfiguration redisConfiguration;
}
```

Now that we have to make this configuration available to the Redis Service, this will be used to start up and setup the redisson client, to do that, first create a field in RedisService so:

```java
final private RedisConfiguration redisConfiguration;
```

You should create a construction that helps to initialise this field as well with an `@Inject` annotation, like so:

```java
@Inject
public RedisService(final RedisConfiguration redisConfiguration){  
  this.redisConfiguration = redisConfiguration;
}
```

The `@inject` annotation notifies Guice to inject the dependency automatically for us. If you are familiar with Spring boot, this is analogous to using the `@Autowired` annotation. Now build and run:

It should throw and error while trying to run, Nothing to worry about, google guice inject dependencies by specifying how you want to bind them, guicey does this autobinding by scanning your project path for Managed classes, Resources that have `@Path` annotations, HealthChecks, Filters annotated with `@WebFilter`, `@WebListener`, `@WebServlet` etc. But for other classes, you have to inform guicey how you want the binding to happen. In other to do that, create a class subclass of `AbstractModule`, and teach Guice how to create your unmanaged dependency as shown below:

```java
package com.opefago.di.config;
import com.google.inject.AbstractModule;
import com.google.inject.Provides;

public class ConfigModule extends AbstractModule {  
  @Provides  
  public RedisConfiguration getRedisConfiguration(AppConfiguration configuration){    
    return configuration.getRedisConfiguration();  
  }

  @Override  protected void configure() {    
    super.configure();  
  }
}
```

The next step will be to register this Module Class so that Guice is aware of it, modify your initialise method in the main class to like like so:

```java
@Override
public void initialize(Bootstrap<AppConfiguration> bootstrap) {
  bootstrap.setConfigurationSourceProvider(      
    new SubstitutingSourceProvider(bootstrap.getConfigurationSourceProvider(),          
                                   new EnvironmentVariableSubstitutor(false)));
  
  bootstrap.addBundle(
    GuiceBundle.builder().enableAutoConfig(getClass().getPackage().getName()).modules(
      new ConfigModule()).build()
  );
}
```

Now build and run the project again, and everything should work out of the box.
With this, we have google guice properly set up and hopefully, understand the basics and can work around almost any challenge. I won’t go into the project details, to see the project in its entirety, visit the repository [here](https://github.com/opefago/dropwizard-guice-di). Hope this material has been of help.

**Conclusion**

Google guice is a very powerful tool for Dependency Injection as well as for Speeding up development, we have only scratched the surface here, Dropwizard though lightweight is also a very good framework for API development projects, with google guice, tons of boilerplate can be avoided.

**References**

1. Design Patterns Explained – Dependency Injection with Code Examples* (2018). Available at: https://stackify.com/dependency-injection/ (Accessed: 9 December 2020).
2. Guide to Google Guice | Baeldung* (2020). Available at: https://www.baeldung.com/guice (Accessed: 22 December 2020).