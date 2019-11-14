# spring-boot-angular-maven-build

## Architecture Overview / Prerequisites

Java 8 - Download and install the JDK 8.  
Node.js - Download and install  
Angular CLI - used to bootstrap the Angular application. You can install it using npm as a default package manager for the JavaScript runtime environment Node.js

In the example that I’m detailing below, we used Spring Boot 2.2.1  RELEASE version.


## Define project modules

The first step of the implementation is to create a multi-module Spring Boot 
application. Initially we create a parent module **spring-boot-angular-maven-build**
which will contain the **backend** module and the **frontend** module.

Assuming that you created the **backend** and **frontend** projects in the 
specified modules.
 * **backend** - will contain the backend project
 * **frontend** - will contain the frontend project

Then, we need to create the modules using Maven Build Tool. We add the modules in the main **pom.xml**

```
<modules>
   <module>backend</module>
   <module>frontend</module>
</modules>
```
Also here we have to specify, the packaging to serve as a container for our sub-modules

```
<packaging>pom</packaging>
```

## Implementing the back-end side

In the **backend** module we implement the parent section

```
<parent>
   <groupId>com</groupId>
   <artifactId>frakton</artifactId>
   <version>0.0.1-SNAPSHOT</version>
</parent>

<artifactId>backend</artifactId>
<version>0.0.1-SNAPSHOT</version>
```
Next, we implement the Maven Resources Plugin. This plugin is used to execute our
generated **frontend** module. In the output directory section we select the 
project build directory and the resources from our **dist/frakton** generated folder. 
In the snippet below we have added also some plugins such as:

* maven-failsafe-plugin - is used and designed for integration tests
* maven-surefire-plugin - is used to run unit tests
* spring-boot-maven-plugin - This plugin provides several usages allowing us to package executable JAR or WAR archives and run the application.

```
<build>
   <plugins>
       <plugin>
           <artifactId>maven-resources-plugin</artifactId>
           <executions>
               <execution>
                   <id>copy-resources</id>
                   <phase>validate</phase>
                   <goals><goal>copy-resources</goal></goals>
                   <configuration>
                       <outputDirectory>${project.build.directory}/classes/resources/</outputDirectory>
                       <resources>
                           <resource>
                               <directory>${project.parent.basedir}/frontend/dist/frakton/</directory>
                           </resource>
                       </resources>
                   </configuration>
               </execution>
           </executions>
       </plugin>
       <plugin>
           <groupId>org.apache.maven.plugins</groupId>
           <artifactId>maven-failsafe-plugin</artifactId>
       </plugin>
       <plugin>
           <groupId>org.apache.maven.plugins</groupId>
           <artifactId>maven-surefire-plugin</artifactId>
       </plugin>
       <plugin>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-maven-plugin</artifactId>
       </plugin>
   </plugins>
</build>
```

After implementing the plugins section, we need to include the **frontend** 
dependency. This will make sure the **frontend** is included in the final
executable JAR.

```
<dependencies>
   <dependency>
       <groupId>com</groupId>
       <artifactId>frontend</artifactId>
       <version>${project.version}</version>
       <scope>runtime</scope>
   </dependency>
 <!-- Other Dependencies -->
</dependencies>
```

## Implement front-end side

Next, we need to implement the pom.xml in **frontend** by adding the parent section:

```
<parent>
   <groupId>com</groupId>
   <artifactId>frakton</artifactId>
   <version>0.0.1-SNAPSHOT</version>
</parent>

<artifactId>frontend</artifactId>
<version>0.0.1-SNAPSHOT</version>
```

To execute some of npm commands we need the **frontend-maven-plugin**.
This plugin comes with a set of built-in commands which we can use for
triggering npm commands

``` 
<build>
<plugins>
   <plugin>
       <groupId>com.github.eirslett</groupId>
       <artifactId>frontend-maven-plugin</artifactId>
       <version>1.7.6</version>
       <configuration>
           <workingDirectory>./</workingDirectory>
           <nodeVersion>v10.16.0</nodeVersion>
           <npmVersion>6.10.2</npmVersion>
       </configuration>
       <executions>
           <execution>
               <id>install node and npm</id>
               <goals>
                   <goal>install-node-and-npm</goal>
               </goals>
           </execution>
           <execution>
               <id>npm install</id>
               <goals>
                   <goal>npm</goal>
               </goals>
           </execution>
           <execution>
               <id>npm run build</id>
               <goals>
                   <goal>npm</goal>
               </goals>
               <configuration>
                   <arguments>run build</arguments>
               </configuration>
           </execution>
       </executions>
   </plugin>
</plugins>
</build>
```

In the configuration tag, we implement the working directory and we select 
the Node and Npm versions.

Also, in the build section we need to define the resources by specifying the 
directory. The **dist/frakton** folder is used to be placed inside the build output.

```
<resources>
   <resource>
       <directory>./dist/frakton</directory>
       <targetPath>static</targetPath>
   </resource>
</resources>
```

## Testing back-end and front-end

After completing the configurations steps, we make sure our instances are working correctly before the build. First we can run the backend project by using:
**mvn spring-boot:run**

*Also make sure you execute the command inside the backend module. 

Next. we can start our Angular project using:
**ng serve**

## Build project

If everything seems to work correctly we can build the project using:
**mvn clean install**

*Make sure you are executing the command in the spring-boot-angular-maven-build parent module*

After building the application, in the backend is generated the target/
folder which contains the jar: backend-0.0.1-SNAPSHOT.jar 
And in the **frontend** is generated the dist/ folder and node_modules.
If you want to run the executable JAR, open terminal and add:

java -jar backend-0.0.1-SNAPSHOT.jar

## Allow Angular to handle routes

After running the JAR file, when accessing https://localhost:8080
we will have a Whitelabel Error Page.

This happens because Angular by default all paths are supported and accessible
but with the usage of Maven, Spring Boot tries to manage paths by itself. 
To fix this, we need to add some configurations. Create a package **config/**
and create **WebConfig.java**. This class has to implement **WebMvcConfigurer** and 
to use **ResourceHandlers**.

``` 
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
  
   registry.addResourceHandler("/**")
           .addResourceLocations("classpath:/static/")
           .resourceChain(true)
           .addResolver(new PathResourceResolver() {
               @Override
               protected Resource getResource(String resourcePath, Resource location) throws IOException {
                   Resource requestedResource = location.createRelative(resourcePath);
                   return requestedResource.exists() && requestedResource.isReadable() ? requestedResource
                           : new ClassPathResource("/static/index.html");
               }
           });
}
```

The /** pattern is matched by **AntPathMatcher** to directories in the path,
so the configuration will be applied to our project routes. 
Also the **PathResourceResolver** will try to find any resource under the
location given, so all the requests that are not handled by Spring Boot 
will be redirected to *static/index.html* giving access to Angular to manage them.

Next, we just build our project and generate the JAR file.

Once the application has started, we will be able to see the welcome page.
