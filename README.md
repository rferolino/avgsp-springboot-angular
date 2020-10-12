# spring-boot-angular-maven-build

## Architecture Overview / Prerequisites

Java 11 - Download and install the Open-JDK 11+.  
Node.js - Download and install  
Angular CLI - used to bootstrap the Angular application. You can install it using npm as a default package manager for the JavaScript runtime environment Node.js

In the example that I’m detailing below, we used Spring Boot 2.2.1  RELEASE version.


## Define project modules

The first step of the implementation is to create a multi-module Spring Boot 
application. Initially we create a parent module **spring-boot-angular-maven-build**
which will contain the **backend** module and the **frontend** module.

Assuming that you created the **backend** and **frontend** projects in the 
specified modules.
 * **services** - will contain the backend project
 * **ui** - will contain the frontend project

Then, we need to create the modules using Maven Build Tool. We add the modules in the main **pom.xml**

```
<modules>
   <module>services</module>
   <module>ui</module>
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
   <artifactId>avgsp</artifactId>
   <version>0.0.1-SNAPSHOT</version>
</parent>

<artifactId>backend</artifactId>
<version>0.0.1-SNAPSHOT</version>
```
Next, we implement the Maven Resources Plugin. This plugin is used to execute our
generated **frontend** module. In the output directory section we select the 
project build directory and the resources from our **dist/avgsp** generated folder. 
In the snippet below we have added also some plugins such as:

* maven-failsafe-plugin - is used and designed for integration tests
* maven-surefire-plugin - is used to run unit tests
* spring-boot-maven-plugin - This plugin provides several usages allowing us to package executable JAR or WAR archives and run the application.

```

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
To fix this, we need to add some configurations. 

``` 
public class AvgStationPortal {

    @Value("${rest.api.base.path}")
    private String restApiBasePath;
    @Value("${cors.allowed.origins}")
    private String[] corsAllowedOrigins;

    public static void main(String[] args) {
        SpringApplication.run(AvgStationPortal.class, args);
    }

    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/**")
                        .allowedMethods("*")
                        .allowedOrigins(corsAllowedOrigins);
            }
        };
    }

    @Bean
    public WebMvcRegistrations webMvcRegistrationsHandlerMapping() {
        AvgStationPortal application = this;
        return new WebMvcRegistrations() {
            @Override
            public RequestMappingHandlerMapping getRequestMappingHandlerMapping() {
                return new RequestMappingHandlerMapping() {

                    @Override
                    protected void registerHandlerMethod(Object handler, Method method, RequestMappingInfo mapping) {
                        Class<?> beanType = method.getDeclaringClass();
                        RestController restApiController = beanType.getAnnotation(RestController.class);
                        if (restApiController != null) {
                            PatternsRequestCondition apiPattern = new PatternsRequestCondition(application.restApiBasePath)
                                    .combine(mapping.getPatternsCondition());

                            mapping = new RequestMappingInfo(mapping.getName(), apiPattern,
                                    mapping.getMethodsCondition(), mapping.getParamsCondition(),
                                    mapping.getHeadersCondition(), mapping.getConsumesCondition(),
                                    mapping.getProducesCondition(), mapping.getCustomCondition());
                        }

                        super.registerHandlerMethod(handler, method, mapping);
                    }
                };
            }
        };
    }
}
```

The /** pattern is matched by **AntPathMatcher** to directories in the path,
so the configuration will be applied to our project routes. 
Also the **PathResourceResolver** will try to find any resource under the
location given, so all the requests that are not handled by Spring Boot 
will be redirected to *static/index.html* giving access to Angular to manage them.

Next, we just build our project and generate the JAR file.

Once the application has started, we will be able to see the welcome page.
