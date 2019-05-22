# My Revature Notes

# Spring Web MVC
https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html

Spring's web framework is built on the Servlet API and deployed on its own embedded Servlet container. Centered around its Dispatcher Servlet, Spring MVC handles much of the complexity of the request-response cycle.

## MVC Design Pattern
- Model: The data being passed, rendered, and manipulated
- View: What will be displayed, usually as html
- Controller: Handles logic, routing

## Spring MVC Request-Response Flow
1. Request sent to `DispatcherServlet`
1. `DispatcherServlet` calls `HandlerMapping` for help with request URI
1. `HandlerMapping` looks up the handler for the URI, a registered `Controller` which is returned to the `DispatcherServlet` and called
1. `Controller` is the entry-point for an event in and out of the rest of the program
1. `Controller` returns a `View` name & `Model` to the `DispatcherServlet`
1. `DispatcherServlet` consults `ViewResolver` to interpret `View` name as a template file and weave the `Model` into the response body
1. Response sent to client


## DispatcherServlet
Spring MVC's front controller has its own WebApplicationContext, allowing it to handle more bean scopes than singleton and prototype. It manages Controllers, HandlerMapping, ViewResolver, and all other components.

## HandlerMapping
While configurable using `RequestMappingHandlerMapping` objects, it can be simply enabled using a `<mvc:@annotation-driven/>` element tag in configuration, allowing for component scanning to automatically register all @Controller and similar beans along with their mappings. It is responsible for routing requests to specific methods within these controllers.

## Controllers
A `@Controller` stereotype annotation registers a class as a library of methods mapped to URI paths to handle requests. Several related annotations help to further specify these requests and their expected responses.

- `@RequestMapping` specifies the URI with attributes like value for the path and method for the HTTP verb, and can be defined at the class or method level.
- `@GetMapping` is a shorthand form of a `@RequestMapping` with GET assumed as its method. Also has siblings in `@PostMapping` and similar annotations.
- `@RequestParam` can be used on method parameters to bind form or query attributes to arguments.
- `@PathVariable` can be used on method parameters to bind URI path variables to arguments.
- `@ResponseBody` tags a method (or all methods of a class) to write their return objects directly to the response body, skipping the ViewResolver entirely.

## ViewResolver
ViewResolvers handle server-side view resolution for static HTML/CSS/JS files, or rendering for dynamic templates like JSPs or Thymeleaf files. 

# AOP - Aspect Oriented Programming
https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop

In OOP, the unit of modularity is the object or class, but it doesn't resolve some issues of tightly coupled code that is only tangentially related to your business logic:
- Logging
- Exception Handling
- Configurations
- Security/Validation
- Transactions
- Tests

In other words: cross-cutting concerns.

In AOP, the unit of modularity is the aspect, otherwise known as a cross-cutting concern. These are code snippets or statements that can be injected into an application which is decoupled from the business logic.

# Spring AOP
With Spring we can define aspects as a class that has several methods which will act as interceptors to other methods in our application. These interceptors are known as advices which will take certain types of actions.
 - **Advice**: Action to be taken at Join Point
 - **Join Point**: Method where code will be injected
 - **Pointcut**: expression which specifies join points where advices will be applied

## Advice
Action that occurs at the joinpoint which has several types:
- Before: preceeds join point, can't interfere with the joinpoint unless exception is thrown
- After - proceeds join point (After-Finally)
    - After-returning: proceeds normal execution of joinpoint
    - After-throwing proceeds if exeption is thrown
- Around - the most powerful advice, acts both before and after, and requires that the developer calls JoinPoint.proceed() to continue join point execution.

## Pointcut
Each advice will have an expression next to their Advice annotation which determines what to listen for in the stack.
- Target: object or method being advised
- AOP Proxy from Spring AOP framework will intercept calls to target and delegate appropriate advice

```java
@Aspect
@Component
public class AnimalAdvisor {
    @Before
    public void beforeGetter(execution(* get*(..)) {
        System.out.println("This prints before every getter is run");
    }
}
```

Examples
- Execution of any public method
> execution(public * *(..))
- Execution of any method starting with 'set'
> execution(* set*(..))
- Execution of any method defined within AnimalService interface
> execution(* com.revature.service.AnimalService.*(..))
- Execution of any method defined within com.revature.demo package
``` execution(* com.revature.demo.*.*(..))```
- Now in subpackage
``` execution(* com.revature.demo..*(..))```
- Within a package
> within(com.revature.demo.*)

# Design Patterns: Dependency Injection
Some classes require another class, a dependency. While the dependency can be defined in the dependant class's own constructor, this introduces several problems such as tightly coupling the two classes and introducing difficulties during unit testing. To solve this, we can inject the dependency as an argument into the dependant class's constructor or setter.

```java
public class Order {
    private Customer customer;

    // Constructor Injection
    public Order(Customer customer) {
        this.customer = customer;
    }

    // Setter Injection
    public setCustomer(Customer customer) {
        this.customer = customer;
    }
	
	public void receipt() { /* Business logic */ }
}
```

Unfortunately, `Order` needs to lookup its Customer dependency elsewhere. You'll need to construct dependencies recursively:
```java
public class App {
	public static void main(String[] args) {
		// Factory logic
		Customer customer = new Customer();
		Order order = new Order(customer);
		
		// Business logic
		order.receipt();
	}
}
```

We can let Spring's IoC Container handle the factory logic for us instead.

# Spring Framework
Documentation: https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html
Spring Framework is a highly modular Application Framework built upon an IoC Container. It offers similar features to JavaEE's EJBs and can interface well with many other specifications such as JPA. Spring can be configured and deployed without the need for an application server with only a few modules, or it can become a self-executing server in the form of a Spring Boot project.

## Inversion of Control (IoC) Container
Spring implements inversion of control through its Core IoC Container, which manages object lifecycles and automatic dependency injection. An object managed by the container, known as a **Spring Bean**, is instantiated, deployed when requested, and destroyed automatically. Any dependencies required by the bean are also injected through constructor arguments, arguments to a factory method, or properties set on the object instance after its construction. The IoC Container is known today as the Application Context, and originally called the Bean Factory.

The goal of an IoC Container is to decouple execution from configuration (business logic from factory logic). By separating these concerns, an application's code base becomes more modular, loosely coupled, with less focus on how code will be implemented and more on its business logic.

The phrase 'inversion of control' refers to this process of configuring dependencies for a bean, rather than the bean controlling the process for its own dependencies.

## Modules
The IoC container is made up of the spring-core, spring-beans, and spring-context modules, but other dependent modules such as spring-aop are commonly used as well. The container can be further extended with the inclusion of other Spring frameworks such as spring-mvc, spring-data, spring-security, spring-cloud, and much more.

## Configuration
While Spring Boot consolidates many modules and follows a 'convention over configuration' approach, most Spring modules can be individually configured either through an XML file or through Java annotations. Spring Boot can load properties files and also offers native support for YAML.

Many modules provide their own preconfigured beans, but custom beans can also be configured by registering them in the XML configuration file or through annotations which are then scanned by the container. Once all beans and configuration options are gathered, an Application Context is created to act as the IoC container for a program.

The Application Context is an interface with several factories to build one according to the manner of configuration, such as ClassPathXmlApplicationContext or FileSystemXmlApplicationContext for XML files, or XmlWebApplicationContext for Spring MVC built on Tomcat. There is also AnnotationConfigApplicationContext for a configuration class.

## BeanFactory vs ApplicationContext
The Bean Factory was the original interface for Spring, and has been superceded by the more capable Application Context. Bean Factory was a sophisticated implementation of the factory design pattern which lazily and programmatically instantiate beans as singletons.

Application Context is an extension of the Bean Factory, eagerly instantiating beans and capable of defining several different scopes besides singleton.

## Bean Lifecycle
Source: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/BeanFactory.html
Spring's container handles the lifecycle of a bean through a complex series of steps. In general, the setup phase involves instantiation of the empty object, handling of its dependencies, initialization of properties and default values, and any custom initialization methods before the bean is ready for use within the program. The teardown phase dereferences the bean when it passes out of scope (or the container is itself shutdown), but also calls any custom destroy methods along the way.

Simplified Lifecycle:
	▪ Instantiate Bean
	▪ Populate Bean (Inject Dependencies)
	▪ Set awareness of context values
	▪ BeanPostProcessor
	▪ (Optional) Custom Init method
	▪ Bean is ready for use! (Bean Mitzvah)
	▪ destroy()/custom destroy method (when container is shut down)
        
As a rule, we do not need to interfere with the lifecycle, but Spring provides several callback methods to customize it in subtle ways. We can implement the InitializingBean/DisposableBean interfaces and override their afterPropertiesSet/destroy methods, or we can define our own custom init and destroy methods in XML configuration or through @PostConstruct/@PreDestroy annotations.

## Bean Scopes
A bean has several scopes, two of which are available to a basic Application Context while the rest are usually seen in a Spring web program.

| Scope | Description |
|------|------|
| singleton | (Default) Scopes a single bean definition to a single object instance for each Spring IoC container.|
| prototype | Scopes a single bean definition to any number of object instances.|
|request | Scopes a single bean definition to the lifecycle of a single HTTP request. That is, each HTTP request has its own instance of a bean created off the back of a single bean definition. Only valid in the context of a web-aware Spring ApplicationContext.
| session | Scopes a single bean definition to the lifecycle of an HTTP Session. Only valid in the context of a web-aware Spring ApplicationContext.|
| application | Scopes a single bean definition to the lifecycle of a ServletContext. Only valid in the context of a web-aware Spring ApplicationContext. |
| websocket | Scopes a single bean definition to the lifecycle of a WebSocket. Only valid in the context of a web-aware Spring ApplicationContext. |

The most important are Singleton and Prototype for most Spring applications. Singleton is the default scope where there is one bean (of its kind) per container, which is best for stateless objects. This is not to be confused with a proper Java singleton which has hardcoded scope within a class loader. A Singleton scoped bean is cached and returned whenever that named object is requested.

Prototype scope can have any number of instances per bean definition, and instead of caching existing beans a new instance is created for each bean request. This makes it ideal for stateful objects, but Spring no longer manages the full lifecycle for us making it little different from calling the 'new' keyword ourselves.

## Bean Wiring 
Spring can inject beans as dependencies of other beans through Setter or Constructor Injection. Beans can be wired manually through property element attributes in an XML configuration file or through annotations. Dependencies can be registered and referenced either by its bean name or by its type.

By far the most popular and easiest method is 'Autowiring' where Spring will figure out a dependency 'automagically' based on its Stereotype annotation.

### Stereotype Annotations
https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/stereotype/package-summary.html
Component is the most basic stereotype and will work for any and all Spring beans. More specific stereotypes such as Service or Repository mostly add unique exception handling for those purposes but otherwise work the same way as Component.

# Deploy with Maven
>mvn package deploy -Duid="AnypointUsername" -Dpwd="AnypointPassword" -Psandbox

# Continuous Integration

>"Continuous Integration is a software development practice where members of a team integrate their work frequently, usually each person integrates at least daily - leading to multiple integrations per day. Each integration is verified by an automated build (including test) to detect integration errors as quickly as possible. Many teams find that this approach leads to significantly reduced integration problems and allows a team to develop cohesive software more rapidly." 
<div style="text-align: right">~Martin Fowler</div>

DevOps culture aims to minimize the barriers between developers and operations as a whole, but it also streamlines the development process itself through Continuous Integration (CI). 

When working with several developers on a single code repository, safely integrating many changes from each developer is a challenge. Too few integrations and the code base may introduce bugs or become broken without any clear insight into when and how it happened. Each developer should not only be responsible for maintaining up-to-date code on their development environments, but also be responsible for regularly committing new code that passes local unit tests.

## Automated builds
As developers integrate new code, an automated build on a server separate from each of their development environments should trigger. After the build is pulled from the repository, compiled, and tested automatically, the developers should be notified of any and all issues or errors introduced by recent commits. This ensures any problems introduced into the repository are quickly highlighted before becoming buried by the next round of commits.

### Scripts
On a build server, the automated process may be as simple as a scheduled script which pulls the latest code from the repository, builds a new artifact from the changes, and logs and reports on the status of any compiling errors or test failures. The following is an example for a Linux server:

#### `build.sh`
```bash
#!/bin/bash
git pull https://your-repository.git
cd your-repository/
mvn clean package
#Mail log output from build to developers
```
Afterwards, register this script as a `cron` job. This will run the script at a determined interval. The script should be made executable, and the server should have all necessary tools installed.

### CI/CD Tools
Advanced tools like Jenkins, Circle-CI, Travis-CI, and many more simplify the automation process. Whether hosted on your own build server or as a cloud service, they can coordinate SCM, build, deployment, and messaging tools through a user-friendly web interface and have access to a large community of plugins or excellent integration with repository services like GitHub.

# Continuous Delivery
>The essence of my philosophy to software delivery is to build software so that it is always in a state where it could be put into production. We call this Continuous Delivery because we are continuously running a deployment pipeline that tests if this software is in a state to be delivered.
<div style="text-align: right">~Martin Fowler</div>

Just as a team of developers use continuous, automated builds to integrate new changes to their code base rapidly, efficiently, and safely, an operations team benefits from automating a rapid delivery process of that build to the various teams and servers in preparation for deployment.

## Pipelines
The process of moving the build (or rebuilding from source) beyond the initial Continuous Integration build is called Continuous Delivery (CD), and the solution that implements this process is known as the deployment (or build) pipeline.

The first CI bulid, the build triggered by a commit or series of commits by developers, should be done quickly to satisfy CI requirements: quick unit tests and error-free compilations are enough. But afterwards the code base should be made available to other teams to run more extensive testing and performance monitoring.

An example two stage deployment pipeline would have the first stage do the CI compilation and unit testing while the second stage builds on a separate testing server to handle integration and end-to-end behavior testing. Further stages can be added at will, triggered at their own intervals.

Just like with CI, simple scripts can automate the process of building, testing, or deploying to staging servers. Tools such as Jenkins also simplify the process of building pipelines and can coordinate multiple servers with ease.

# Jenkins
Forked from Sun's Hudson project after it was acquired by Oracle, Jenkins is a popular open-source CI/CD tool used throughout the industry to create and manage build pipelines. Jenkins is distributed as a Java `war` package and can be deployed to a Tomcat server or run as a self-executing `war`. The Jenkins dashboard is then accessible through a web application.

## Setup
When Jenkins first runs, it creates a `.jenkins` directory where pipelines and project workspaces will be managed, along with an installation directory elsewhere for managing tools, plugins, and users.

Once Jenkins is up and running, the dashboard can be found at `http://localhost:8080` by default. The Unlock Jenkins page will prevent further access until the autogenerated password is copied from the file location specified on the page or on the console log output when running Jenkins as an executable `war`.

Once unlocked, Jenkins can download several popular plugins and help configure the admin account.

## Plugins
The popularity of Jenkins is in part thanks to its rich plugin community offering many helpful tools for a variety of build actions. On initial setup a variety of popular plugins come bundled with Jenkins and can be installed individually. More plugins can be discovered and installed (or removed).

## Global Tools
Jenkins runs its processes as a special user on its host operating system. In order to work with a variety of tools, it must install and configure permissions for these tools, ranging from command line programs such as Git or Maven to authentication of external services like GitHub or Slack. If certain tools are missing, Jenkins can be configured to automatically install them during a build.

## Users
A default admin account is registered during initial setup. If this admin account creation is skipped, the password will be the autogenerated password used to unlock Jenkins. Other accounts can be created, and permissions can be set by the admin using the security matrix. This is useful for restricting access to your pipelines to developers who may not require it.

## Jobs
A pipeline is defined through a Jenkins Job, a project created and monitored on the Jenkins dashboard. Jobs can be structured in multiple ways using a variety of archetypes. Each job creates a folder with a workspace directory where a bulid can be isolated from other jobs. A job is configured through a graphical wizard interface or programmatically using a Jenkinsfile script (written in Groovy, a JVM scripting language).

Besides the build itself, various pre-build and post-build stages can be defined in order to set environment variables, prepare build tools, poll SCM repositories, and log messages to email or other messaging services like Slack.

## Agents
Jenkins jobs normally run on the master node, the server the Jenkins instance defining the pipeline shares. But distributed builds are possible through a Jenkins Agent, a separate node hosted on a different server and linked to a master node through SSH. A Jenkins master with one or more agents can run a build on these agents thereby freeing up resources on the master for other jobs.

# Example Plugin use: GitHub Hooks
Rather than poll a GitHub repository at regular intervals, Jenkins can be configured to listen to the repository for any updates, then trigger a build.

## On GitHub
On a repository, under "Settings" and "Integrations and Services", the Jenkins (GitHub plugin) service can be added and configured using  `http://<your public DNS>:8080/jenkins/github-webhook/` as the Jenkins hook URL. "Active" should be checked.

Under the user's profile, "Settings", then "Personal access tokens" under "Developer settings", a user access token can be generated and assigned a repository scope. The generated token, an alphanumeric string pattern, can be copied over to Jenkins.

## On Jenkins
In "Manage Jenkins", "Global Tool configuration", The path to Git should be configured. Then Under "Configure System", then "Github", and "Add GitHub Server", the hook can be added. The API URL should be `https://api.github.com` while in "Credentials", then "Add/Jenkins" and "Kind", the "secret text" should be the generated token copied from GitHub. "Test connection" should display a message saying, "Credentials verified for user `<your github name>`... etc".

Then while configuring a project job, "Git" should be chosen under "Source Code Management" specifying the repository URL and branch to build from. Afterwards the "GitHub hook trigger for GITScm polling" under "Build Triggers" should enable the feature for the pipeline.


# Getting started with Mule
[Anypoint Studio](https://www.mulesoft.com/lp/dl/studio) as a stand-alone IDE requires registration as it includes the enterprise edition of the Mule Kernel (previously known as the Mule ESB Runtime). But, so long as we use Mule 3 or below, we can add Anypoint Studio as a suite of plugins to Eclipse. The [Mule Kernel](https://developer.mulesoft.com/download-mule-esb-runtime) is also available as a stand-alone runtime.

# Anypoint Studio (6) in Eclipse
Official reference documentation is available [here](https://docs.mulesoft.com/studio/6/studio-in-eclipse).

First, we download and extract [Eclipse 4.5.2](https://www.eclipse.org/downloads/packages/release/mars/2). Next under "Help -> Install New Software" enter `http://studio.mulesoft.org/r5/plugin` into the 'Work With' field. Select all desired options and install (be patient, this may take some time).

To add the community edition runtime, after installing Anypoint Studio plugins, enter `http://studio.mulesoft.org/r5/studio-runtimes/` in the 'Work With' field of the "Help -> Install New Software" wizard.

## [Official Hello-World Tutorial](https://docs.mulesoft.com/general/getting-started/build-a-hello-world-application)

# Mule Terminology
## Events
An event source receives a trigger (HTTP, JDBC, FTP, JMS) and create a corresponding mule event object and forwards the processing. Mule events are immutable. Any change to an instance of a mule event results in the creation of a new mule event instance. An event is composed of a **MuleMessage**, **MuleSession**, and **MuleContext**. 

## Messages
**Mule Message** is composed of different parts:
- payload
- properties
- attachments (optional)
- exception payload (optional)

**Message Sources** can be inbound endpoints, pollers, or cloud connectors. Only a single message source is allowed in a flow. A composite message source must be used to allow for multiple inbound endpoints.

**Message Processors** take care of performing all the message handling operations in Mule. They include:
- **outbound endpoints** which dispatch messages to whatever destination you want
- **transformers** which modify messages
- **routers** which ensure messages are disturbed to the right destinations
- **components** which perform business operations on messages

**Message Exchange Patterns (MEP)** define the timely coupling that will occur at a particular inbound or outbound endpoint. 
- **One-way** where no synchronous response is expected from the interaction but will still receive status code of sent message
- **Resquest-response** where a synchronous response is expected so it'll wait for a response before proceeding

## Flows 
A mule application can consist of a signle flow or break up processing into discrete flows and subflows which can be connected together. The latter is usually done in order to break applications into functional modules / error-handlers that can be used over and over again **(DRY)**. Flows and subflows are like function calls where events are passed in as inputs and modified events are what is returned. They can be connected with **Flow Reference components** under the component tab in your Mule Palette and other ways (not important right now).

**Flows** typically start with a source, such as a HTTP listener, to trigger their executions. They can be configured to be initially stopped in the Runtime Manager for when you do not want a source to start a them right away. The **Flow Reference component** is used to trigger flows without a source. Individual flows should utilize **DRY** by having specific flows for API requests from a web client, processing the event, returning an appropriate response, etc.

**Subflow** is a scope that groups together a sequence of event processes (like a flow), but do not start with an event source and do not have their own error handling. They can be called from either another flow or sublfow through a **Flow Reference component**. The whole mule message is passed & its context are passed into a subflow. The complete context of the result of the subflow is passed back to the main flow.

**Private Flows** are another type of reusable flow. They can define a different processing / exception strategy from the calling flow, are materialized at runtime meaning they have specific statistics and debug properties attached to them, and can be controlled & monitored independently .

## Building Blocks
**Connectors**
- Endpoint
- Send and receive 
- Host, address, port

**Components**
- General
- Transform message
- Logger
- Flow reference
- Scripts
- Web services

**Transformers** (components used to set or remove part of the Mule event)
- Transforms
- Set payload
- Set variable
- Remove variable

**Filters**
- 12+ included
- and, or, not
- Custom

**Routers**
- Message flow
- Splitting
- Re-sequencing
- Aggregation

**Scopes**
- Wrappers
- Processing blocks

**Exception Strategies**
- Errors 
- Faults
- Strategies

# API Life Cycle
- Analysis: define API business objective

- Development: define technical requirements (methods, protocols, access, scalability, operations, hosting, etc)

- Operations: release your Minimal Viable Product, foster a user-base, fix, re-fix

- Retirement: deprecation of the API, can be due to limited use, outdated plugins, lack of support, security concerns, etc.


# Soap-Demo
Contract Last example generating a JAX-WS Soap service. WSDL generated and found at:
http://localhost:8080/hello?WSDL

Example message to sayHello() method:
```xml
<Envelope xmlns="http://schemas.xmlsoap.org/soap/envelope/">
    <Body>
        <sayHello xmlns="http://soap.revature.com/">
            <name xmlns="">YourNameHere</name>
        </sayHello>
    </Body>
</Envelope>
```

# Soap
  * “Expose” and “Consume” Web Services
  * Simple Object Access Protocol
  * Protocol for XML based communication across the Internet
  * Platform and language-independent
  * Similar: Corba, Dcom, Java RMI
  * SOAP is pure XML and therefore language agnostic
  * Not tied to a specific transport protocol: HTTP, SMTP, FTP, MSNQ, IBM MQSeries, etc.

### SOAP w/HTTP
  * Messages sent within Http POST Requests
  * Http must set content type to text/xml

### SOAP Message
  * Envelope – (mandatory), defines start/end of message
  * Header – (optional), optional attributes to be used when processing message
  * Body – (mandatory), XML data with message to be sent
  * Fault – (optional), describes errors that may have occurred when processing
  * Defined in default namespace
  * for SOAP – www.w3.org/2001/12/soap-envelope
  * for encoding and datatypes – www.w3.org/2001/12/soap-encoding
  * Envelope packaging contents of message
  * Only one body element allowed
  * Changing SOAP version requires change of envelope
  * SOAP Header/Body != HTTP Header/Body: entire message goes inside Http Req/Resp body
  #### Message Structure
  ```xml
    <xml version=“1.0”?>
    <soap-env:Envelope xmlns=soap-env=”www.w3.org/2001/12/soap-envelope” soap-env:encodingStyle=”www.w3.org/2001/12/soap-encoding”>
      <soap-env:Header>
      </soal-env:Header>
      <soap-env:Body>
        <soap-env:Fault>
        </soap-env:Fault>
      </soap-env:Body>
    </soap-env:Envelope>
  ```

  #### Faults
  * One fault block per message
  * Optional
  * Success: 200-299 (HttpBinding)
  * Soap Fault: 500-599
  * Elements
    * `<faultCode>`
    * `<soap-env:versionMismatch>` - invalid namespace for env
    * `<soap-env:mustUnderstand>` - immediate child of header element not understood
    * `<soap-env:client>` - message was incorrectly formed
    * `<soap-env:Server>` - error with server
    * `<faultString>`
    * `<details>`
    
### WSDL
  * Web Service Description Language
  * XML file describing everything about the service
  * “Contract” or “Endpoint”
  * Like an interface
  * Contract first vs contract last
  * Two ways of creating a soap service
  * Did you write WSDL first, or Implementation (Java code) first?
  * Important WSDL Elements (definitions):
  * name (optional)
  * targetNamespace – logical namespace for info about service
  * xmlns – default wsdl namespace, http://schema.xmlsoap.org/wsdl
  * All WSDL elements go in this namespace
  * xmlns:xsd, xmlns:soap
  * Specifies soap-specific info and datatypes
    * types port message porttype operation (definition) binding service
    * types – any complex datatype used in document (not necessary if only simple types)
    * port – specify single endpoint as address for binding
    * message – define data elements for each operation (method params, return values)
    * porttype – defines operations that can be performed and the messages involved
    * operation – abstract description of action supported by service
    * binding – specify protocol and data format for operations and messages
    * service – specify port address(es) of binding
    
### Jax-WS – Java API for XML Web Services
  * This is to set up service providers + consumers
  * Annotation-based
  * Supported by open source framework Apache CXF (also supports other protocols)
  * Jax-B – Java Architecture for XML Binding
  * Marshalling – convert Java obj to XML file
  * Unmarshalling – convert XML to Java obj
  * WSDL Soap Binding styles
  * Use to translate WSDL binding to soap message body
  * Model: Literal (DOCUMENT) vs Encoded (RPC)
  * DOCUMENT – you define XML structure of message body (“message-oriented”)
  * RPC – request body must contain operation name and method parameters
  * Literal: contents conform to user-defined xsd
  * Encoded: uses xsd datatypes but body doesn't need to conform to user-defined xsd

# HTTP/HTML
Hypertext Transfer Protocol & Hypertext Markup Language, the two main technologies for a web browser. HTML provides a structure for static content on web pages. HTML documents are organized by element tags, some with attributes. Browsers don't display the element tags, but instead parse them to render content.

<html>
    <body>
        <p>Hello there!</p>
        <a src="www.google.com">Google</a>
    </body>
</html>

HTTP is all about sending and receiving documents. A document is abstracted from HTTP packet that has information like metadata, contents, status codes.

## HTTP Status Codes
- 100-199: INFO
- 200-299: OK/Success
- 300-399: Redirect
- 400-499: Error/Request Incomplete
- 500-599: Server Error

## HTTP Methods (Verbs)
 - GET: retrieves information, sends info in the URL itself, empty body
>localhost:8080/ExampleContext/example?name="bob"

 - POST: Send info as text or binary in a body
 - PUT
 - DELETE
 - PATCH
 - UPDATE
 - TRACE
 - HEAD
 - OPTIONS

# JavaEE
Java Enterprise Edition, this is a community driven collection of Specifications, APIs and Frameworks which provide enterprise functionality. We will start with Java Servlets, a JavaEE API for communicating between a Java code base and HTTP.

Found in `javax.servlet` and `javax.http` the Servlet API provides a Servlet Interface which is implemented by the GenericServlet Abstract Class, then the HTTPServlet Abstract Class, and then finally allows you to extend the HTTPServlet.

In a normal servlet process:
- The client sends an HTTP Request to a (Servlet) Web Container, which is a Java Server that is running your application. We will be using Apache Tomcat, which is Apache's lightweight implementation of the JavaEE Server and Servlet Container spec.
- Your Tomcat server will have your application in its webapps folder (as a war, usually), and this application will have the following:
    - *src/*: for the servlet and other business logic code
    - *webapp/*: for any hosted HTML-related files, as well as:
    - **Deployment Descriptor**: commonly a *web.xml* file that maps incoming requests to your Servlets.
- Java application on Tomcat will handle Request as well as the Response by creating Java objects to model the Request/Response.

# Servlet

## Servlet Lifecycle
The Tomcat server will handle the control flow of your application through a servlet lifecycle process.
1. When a request comes in, the container instantiates any appropriate servlets
1. Once instantiated the container initializes the servlet by invoking its `init()` method
1. After initialization (or existing servlet is found), the container invokes `service()` to process the request.
    1. public void service (ServletRequest req, ServletResponse resp) {...} ->
    1. protected void service (HttpServletRequest req, HttpServletResponse resp) {...} ->
    1. protected void doGet (HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {...} //You write this
1. Response object is returned to client
1. When the container shuts down or needs to conserve memory or just because a servlet's stated lifespan is reached, the container will call the `destroy()` method of your servlet.

tl;dr Container calls init() once, service() many times, and eventually destroy() once.

## WAR
JEE web applications are usually packaged as `.war` files, a web archive similar to a `.jar` but with some minor changes to folder heirarchy. A `src/main/webapp` folder becomes the root directory, and the archive is usually hosted on a Java web server such as **Apache Tomcat**. Be sure to set the `packaging` property in your `pom.xml` to `war`:
#### pom.xml
```xml
...
<packaging>war</packaging>
...
```

## Dependencies
To begin using Java Servlets, include the following dependency to your Maven `pom.xml`:
#### pom.xml
```xml
...
<groupId>javax.servlet</groupId>
<artifactId>servlet-api</artifactId>
<version>2.5</version>
...
```
`Servlet` is a JEE specification which is not included in Java's standard library, so it must be imported or the project must use a JDK with JEE libraries included already.

## Deployment Descriptor
A Java server like Tomcat will deploy a web archive and register all servlets according to the *deployment descriptor* found in `src/main/webapp/WEB-INF/web.xml`:
#### web.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" version="2.5">

...

</web-app>
```
Servlets, Filters, Context parameters, and other configurations are declared and defined in the `web.xml` 

## Servlet Mapping
Servlet classes are registered in the `web.xml` by assigning a name to both a url mapping and the fully qualified class name of the Servlet:
#### web.xml
```xml
...
<servlet>
    <servlet-name>myServlet</servlet-name>
    <servlet-class>servlets.MyServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>myServlet</servlet-name>
    <url-pattern>/myServlet</url-pattern>
</servlet-mapping>
...
```
This will provide access to the servlet through its url mapping, in the above case at `http:[hostname]:[port]/[app-context]/myServlet`

## Servlet
The Servlet API provides the `HttpServlet` abstract class which can be extended to define your own custom behavior:
#### MyServlet.java
```java
package servlets;

import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class MyServlet extends HttpServlet {
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp)
		throws ServletException, IOException {
            // Implements GET behavior
	}
	
	@Override
	protected void doPost(HttpServletRequest req, HttpServletResponse resp)
		throws ServletException, IOException {
            // Implements POST behavior
	}
}
```
The `HttpServletRequest` object provides useful methods such as `getParameter(String name)` which returns the value of a Query or post body parameter. `HttpServletResponse` provides a `getWriter()` method which returns a `PrintWriter` object to append data to the response body.

## Servlet 3+
More recent versions of the Servlet API offer convenience annotations that allows configuration in a decorator over the Servlet class itself, leaving the `web.xml` blank for the most part:

#### pom.xml
```xml
...
<groupId>javax.servlet</groupId>
<artifactId>javax.servlet-api</artifactId>
<version>3.0.1</version>
<scope>provided</scope>
...
```

#### MyServlet.java
```java
package servlets;

import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet("/myServlet")
public class MyServlet extends HttpServlet {
	protected void doGet(HttpServletRequest req, HttpServletResponse resp)
		throws ServletException, IOException {
            // Implements GET behavior
	}
}
```

# Java
Java is an object-oriented programming language and a platform developed by Sun Microsystems (eaten by Oracle). Using the principle of **WORA** (Write Once, Run Anywhere), a Java application can be compiled and executed on any platform supported by Java. Flexible, popular, and well-supported, Java has helps developers write scalable client-server web applications, desktop and mobile applications, and frameworks and libraries.

## Features
- **Platform independence**: The compiler converts source code to bytecode, then the JVM executes that bytecode. While each operating system has their own JVM implementation, so every JVM can execute bytecode regardless of origin of said code.

- **Clear, verbose syntax** Java has C-like syntax like C++ and C#, many syntax elements of the language are readable and widely used in programming. The Java API (Application Programming Interface) is also written using verbose, descriptive naming conventions making it simple to parse large code bases.

- **Multi-paradigm** While Java is object-oriented, it supports multiple paradigms such as imperative, generic, concurrent, and functional.

- **Garbage collection** The JVM performs automatic memory deallocation of unused objects at runtime. Programs are written without the need for complex memory management.

- **Multithreading** Java supports multithreading at both language and the standard library levels. It allows concurrent and parallel execution of several parts of a Java program.

# Programming and Compiling
Most Java applications only require the **JRE** (Java Runtime Environment). But to write and compile you need the **JDK** (Java Development Kit). While the JRE provides Java's standard libraries and exceptions as well as a JVM, the JDK provides all the above as well as *javac*, the compiler. Java source code is written in text files labeled with *.java* extension. It is then compiled into bytecode in *.class* files by *javac*. Then the bytecode is executed by the JVM, which translates the Java commands into low-level instructions to the operating system.

Since Java 6, all Java programs not run inside a container (such as a Servlet Web Container) start and end with the main method. The class containing the main method can have any name, but the method itself should always be named *main*

```java
class Example {
    public static void main(String[] args) {
        System.out.println("Num args:" + args.length);
    }
}
```

- *public* is a Java access modifier keyword that means the `main` method can be accessed from any method during the program's execution.
- *static* is a Java keyword that means the method can be invoked without creating an instance of the class that contains it, making it a global method.
- *void* is a Java return type keyword that means the method doesn't return any values of any data type.
- *args* is a Java variable of type String array which means the method can take command line arguments as an array of Strings

We can compile this code into a *.class* file of the same name:
>javac Example.java

And to run the resulting `Example.class` file:
>java Example

The `java` and `javac` commands require the full directory path or class path to any source code or binary file respectively. If you have a package `com.demo` in the first line of Example, then you would nest the java file into a `com/demo/` directory and then run:
>javac com/demo/Example.java 

>java com.demo.Example

From here we can add packages and imports, expanding the application into a set of interacting objects. By default, the *javac* compiler implicitly imports several base packages from the standard library.

# Maven 
Build automation and dependency management tool. Once installed, use with the `mvn` command. Allows for a project to be IDE agnostic. See the official Maven project for documentation: http://maven.apache.org/index.html as well as the mvn repository to find available libraries: https://mvnrepository.com/

The minimum `pom.xml` example:
```xml
<project>
	<modelVersion>4.0.0</modelVersion>
	<groupId>YourDomainName</groupId>
	<artifactId>YourProjectName</artifactId>
	<version>0.1.0</version>
</project>
```

## Example commands
Create a new Maven project with the quickstart archetype. Change groupId and artifactId arguments as needed:
>mvn archetype:generate

Or skip the setup and run the generator in one line:
>mvn archetype:generate -DgroupId=com.revature -DartifactId=my-first-app -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

Compile class files into target/classes
>mvn compile

Package project into a jar file in target
>mvn package

Remove target folder and compiled build
>mvn clean

## Object-Oriented Programming
Although Java accommodates several paradigms, OOP is the foundation for most applications. In OOP, a program is organized into objects encapsulating related fields (representing its *state*) and methods (usually to control that state or perform related functions). When defining objects, Java reserves the keyword *class* (not to be confused with the *.class* file extension) which serves as their blueprint. An object in Java represents an instance in memory of a class, and also every class implicitly inherits from the *Object* superclass which provides useful convenience methods such as *equals()* and *toString()*. Java popularized several 'Pillars' of OOP design theory. While the numbers vary between OOP languages, Java focuses on four:

    - *Abstraction* - By simplifying objects to a set of useful features, we hide irrelevant details, reduce complexity, and increase efficiency. Abstraction is important at all levels of software and computer engineering, but essential to designing useful objects. Complicated real-world objects are reduced to simple representations.

    - *Encapsulation* - Objects should group together related variables and functions and be in complete control over them. So the state of an object should only change, if ever, through the object itself. Also known as data hiding, because the object has sole responsibility for its fields, and no outside object or function should interfere.

    - *Inheritance* - Code reuse is an important principle of programming (DRY - Don't Repeat Yourself), and new classes can reuse code from existing ones. This establishes a superclass-subclass (or parent-child) relationship where the derived classes inherit (and sometimes change) fields and methods from its parent.

    - *Polymorphism* - With inheritance, an object of a derived class can be referenced as instances of its parent class. This provides flexibility when invoking inherited methods with varying implementations in derived classes.

## Variables
A value is stored and identified in memory by a variable. Variables have a name that makes it possible to access the value, and a type that defines what sort of value it stores.
```java
int variableName = 64;
String txtVar = "Hello World";
```

## Primitive data types
Java handles two kinds of datatypes: primitives and references. Primitives are variables that store simple values. There are eight in Java.
- Integer types: **byte**, **short**, **int**, and **long** (42)  
- Floating-point types: **float**, and **double** (3.1415)  
- Logical types: **boolean** (true)  
- Character type: **char** ('x')

## Reference types
Reference types store the memory address location of more complex data types in the heap. Reference types include:
- Classes
- Interfaces
- Enums
- Arrays

## Naming variables
- Case sensitivity
- Can only use letters, numbers, and *$* or *_* characters
- Cannot begin with a number
- Cannot be a reserved Java keyword

## Scopes of a variable
A variable's reference will only exist within the context of its declared scope, which is based on the location of its declaration.

- **Static** or class scoped variables are visible to all instances of a related class.
- **Instance** or object scoped variables are visible to only that object instance.
- **Local** or method scoped variables are visible only within a method.
- **Block** or loop scoped variables are visible only within a block statement.

Be aware of *shadowing*: when two variables in different scopes share names.

## Methods
Methods accept a list of arguments known as *parameters* and return some value. They are used to implement repeatable, consistent actions on variable input, much like math functions.
```java
public int myMethod(int a, int b);
public int myMethod(int a);
```

## Constructors
Classes not only define object fields and methods, but how it should be instantiated through special methods called constructors. Constructors must have no return type and share the same name as its class. Java will automatically give you a *noargs* constructor. However, if you define any constructor, you will lose the automatically given constructor.

While a constructor may be *private*, used for singletons, it may not be *final*, *static*, or *abstract*.

## Access modifiers
- **private** - accessible only within the context of that class
- **default** - accessible within the context of a package, has no associated keyword so is set when no modifier is used
- **protected** - accessible to the package, but also to derived child classes outside of the package
- **public** - accessible anywhere

Classes should only be public or default. There are no cascading access levels, and unspecified fields will be default. Subclasses can only change inherited fields to be less restrictive.

# Arrays
## Declaration
Java Arrays are special reference types that store similarly typed data iteratively. A pair of brackets define an array of some data type, and can be written anywhere after the type:
```java
// One-Dimensional Arrays
int[] arrayOne;
int []arrayTwo;
int arrayThree[];

// Two-Dimensional Arrays
int[][] 2DArrayOne;
int 2DArrayTwo[][];
int []2DArrayThree[];
```

## Definition
While Java does not allow direct memory access to its arrays like other languages, they are still of fixed size once defined by the `new` keyword or by an array literal.
```java
// One-Dimensional Arrays
int[] instancedArray = new int[3];
int[] literalArray = {1, 2, 3};

// Two-Dimensional Arrays
int[][] instanced2DArray = new int[3][4];
int[][] literal2DArray = { { 1, 2 }, { 3, 4 }, { 5, 6 } };
```

## Iteration
Java for loops can iterate through arrays like most other languages:
```java
// One-Dimensional Arrays
for (int i = 0; i < arrayOne.length; i++) {
    System.out.print(arrayOne[i]);
}

// Two-Dimensional Arrays
for (int i = 0; i < 2DArrayOne.length; i++) {
    for (int j  =0; j < 2DArrayOne[i].length; j++) {
        System.out.print(2DArrayOne[i][j]);
    }
}

// Foreach loops
for (int i : arrayOne) {
    System.out.print(i);
}
```

## Manipulation
The `java.util.Arrays` class provides various methods for manipulating arrays.

```java
int[] messyArray = {234, 5346, 3, 64};
Arrays.sort(messyArray);
System.out.println(Arrays.toString(messyArray));
```

## Varargs
Varargs is a special parameter that can accept multiple arguments of the same type into a dynamically constructed array, and denoted by an ellipsis (...) instead of brackets. A varargs parameter must be the last or only parameter in a method signature. 
```java
varArgMethod("m", 1, 2, 5, 35, 346, 345, 4634);

...

public static void varArgDemo(String m, int... intArgs) {
    for (int i : intArgs) {
        System.out.print(i);
    }
}
```

# Exception Handling
When an something wrong occurs during execution, the current stack frame will throw an exception. If the exception is not handled, or thrown up the stack to be handled elsewhere, the program will crash. Good exception handling helps a program continue execution. Common issues which can throw exceptions involve stack or heap memory overflow, an array iterating out of bounds, or an interrupted stream or thread.

## Hierarchy
Exception and error objects extend Throwable and are either checked or unchecked.
* Throwable (checked)
    * Exception (checked)
        * RuntimeException (unchecked)
    * Error (unchecked)

Checked exceptions
Unchecked exceptions / Runtime exceptions
Errors
Runtime and unchecked exceptions refer to the same thing. We can often use them interchangeably. 

Checked Exceptions are compile-time issues that must be handled or thrown before the compiler can build, such as `IOException`. Unchecked Exceptions occur at runtime, so the compiler cannot predict them and does not force they be handled. Most unchecked exceptions extend RuntimeException, such as `NullPointerException`. Errors are serious issues and should not be handled, such as `StackOverflowError`.

## Throws
The `throws` keyword re-throws an exception up the stack to the method that called the throwing method. If the exception is continually thrown and never handled, the compiler will be satisfied in the case of checked exceptions but any issues will still break the program.
```java
public void methodThatThrows() throws IOException {
    // throw (singular) will throw a new exception every time.
    throw new IOException();
}

public void methodThatCalls() {
    methodThatThrows(); // IOException must now be handled here, or methodThatCalls() must use throws as well
}
```

## Try-Catch
The most basic form of exception handling is the try-catch:
```java
public void methodThatThrows() throws IOException {
    try {
        throw new IOException();
    } catch (IOException exception) {
        // Do something with the exception
        logger.warn("IOException thrown");
    }
}
```

A try block must be followed by at least one catch (or finally) block, but there can be any number of catch blocks for specific (or broad) exceptions. Catch blocks must be ordered from most specific to least specific Exception objects else later catch blocks catching subclasses of exceptions caught in catch blocks above it will become unreachable code.

Multiple exceptions can also be handled in one catch block:
```java
public void methodThatThrows() throws IOException {
    try {
        throw new IOException();
    } catch (IOException ex1 | ServletException ex2) {
        // Do something with the exception
        logger.warn("IOException thrown");
    }
}
```

## Finally
Try blocks can be followed by one finally block, and can either replace the mandatory single catch block or follow one or more catch blocks. They are always guaranteed to execute, even if no exceptions are thrown, and are useful for closing resources that may be left open in memory due to an interruption from a thrown exception.
```java
public void methodThatThrows() throws IOException {
    try {
        throw new IOException();
    } finally {
        System.out.println("Will always run");
    }
}
```

## Try-with-resources
Declaring and defining a resource - any object that implements AutoCloseable - within a pair of parenthesis after the try keyword removes the necessity of a finally block to close that resource.
```java
public void methodThatThrows() throws IOException {
    try (FileReader fr = new FileReader()) {
        throw new IOException();
    } catch (IOException exception) {
        logger.warn("IOException thrown");
    }
}
```

# I/O
## InputStream/OutputStream -> BufferedReader/BufferedWriter
The JVM can connect to external datasources such as files or network ports. InputStream/OutputStream and its implementations stream this data as an array of bytes whereas Reader/Writer and its implementations wrap InputStream/OutputStream to stream data as a char array. BufferedReader/BufferedWriter wraps Reader/Writer to stream several characters at a time, minimizing the number of I/O operations needed.

```java
BufferedReader br = new BufferedReader(
  new StringReader("Bufferedreader vs Console vs Scanner in Java"));
BufferedReader br = new BufferedReader(
  new FileReader("file.txt"));
BufferedReader br = new BufferedReader(
  new InputStreamReader(System.in));

BufferedReader fbreader = new BufferedReader(new FileReader("input.txt"));
BufferedReader isbreader = new BufferedReader(new InputStreamReader(System.in));
BufferedReader niofbreader = Files.newBufferedReader(Paths.get("input.txt"));

try (BufferedReader reader = new BufferedReader(new FileReader("input.txt"))) {
	return readAllLines(reader);
}

public String readAllLines(BufferedReader reader) throws IOException {
	StringBuilder content = new StringBuilder();
	String line;
	while ((line = reader.readLine()) != null) {
		content.append(line);
		content.append(System.ineSeparator());
	}
	return content.toString();
}
```

# Scanner
BufferedReader provides many convenient methods for parsing data. Scanner can achieve the same, but unlike BufferedReader it is not thread-safe. It can however parse primitive types and Strings with regular expressions. Scanner has a buffer as well but its size is fixed and smaller than BufferedReader by default. BufferedReader requires handling IOException while Scanner does not. Thus, Scanner is best used for parsing input into tokenized Strings.

```java
Scanner sc = new Scanner(new File("input.txt"));
Scanner issc = new Scanner(new FileInputStream("input.txt"));
Scanner csc = new Scanner(System.in);
Scanner strsc = new Scanner("A B C D");
```

# Properties
Load properties as key-value pairs from a file
//app.properties
```
key=value
```

```java
Properties props = new Properties();
props.load(new FileInputStream("app.properties");
String value = props.getProperty("key", "defaultValue");
```

# Security
```java
SecureRandom random = new SecureRandom();
byte[] salt = new byte[16];
random.nextBytes(salt);

// Using SHA
MessageDigest md = MessageDigest.getInstance("SHA-512");
md.update(salt);
byte[] hashedPassword = md.digest(passwordToHash.getBytes(StandardCharsets.UTF_8));

// Using PBKDF2
KeySpec spec = new PBEKeySpec(password.toCharArray(), salt, 65536, 128);
SecretKeyFactory factory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA1");
byte[] hash = factory.generateSecret(spec).getEncoded();
```

# Reflection
Reflection allows one to examine or modify runtime behavior of a program. Java's Reflection API mostly allows introspection of structure, while modifying is only allowed on access modifiers of methods and fields. Many frameworks such as JUnit, Application/Servlet Containers, and Spring use reflection to examine class fields, construct objects, and invoke methods at runtime.

```java
Class<?> c = Class.forName("classpath.and.classname");
Object o = c.newInstance();
Method m = c.getDeclaredMethod("aMethod", new Class<?>[0]);
m.invoke(o);
```

# Generics
When passing objects into methods and data structures, a developer can overload or extend for its specific type or cast the object up and down its inheritance heirarchy. In contrast a generic type improves code reuse and type safety, reducing code by allowing methods and data structures to accept any type without risking dynamic runtime exceptions. Generic type parameters act as placeholders in a method signature while diamond operators specify the type for the compiler to enforce at compile time:

```java
ArrayList<String> list = new ArrayList<>();

public <T> String genericToString(T a) {   
    return a.toString();
}


public <T, E> String genericToStrinCat(T a, E b) {   
    return a.toString() + b.toString();
}
```

The type parameters T and E will be replaced by the compiler through type erasure:
```java
String s1 = genericToString(1);
String s2 = genericToString("Hello", 3.5);
```

# Collections Framework
Java's collections framework implement common data structures for objects.

- **List** is an ordered collection of elements. A user has the ability to place an element anywhere in the list. The elements are accessable by their index. Unlike **Set**, **List** typically allows for duplicate elements such that element1.equals(element2). In addition to duplicates, **List** allow for multiple null elements to be stored.  
  
- **Set** is a collection of non duplicate elements meaning there will never exist a situation where element1.equals(element2). In addition to this, it is implied that there can only exist one null element due to the no duplicates rule.  

- **Queue** is a collection of elements who in essence cannot be iterated, instead the **Queue** follows the **FIFO** (First In First Out) rule. When an element is added to the **Queue** it is placed at the back and when an element is pulled it is pulled from the from the front (index :0).  
  
- **Deque** extends **Queue** but augments the ability to insert and remove elements at either end. The name is short for "Double Ended Queue" and is pronounced "Deck".  
  
- **Map** is an interface which stores data with a key. A map cannot contain duplicate keys; each key can map to at most one value. **Map** can be visualized like a dictionary where only one word is paired with one definition.  

# Comparable vs Comparator
Comparable is a functional interface used to define a 'natural ordering' between instances of a class, commonly used by sorted collections such as TreeSet.

Comparator is another functional interface used in a dedicated utility class that can compare two different instances passed to it. It can be passed to a sort method, such as Collections.sort() or Arrays.sort(), or to sorted collections.

For example, to automatically sort a TreeSet of type Person according to age. We can either make the object class comparable or pass the constructor a comparator.

## Comparable
```java
class Person implements Comparable<Person>{
	int age;
 
	Person(int age) {
		this.age = age;
	}
 
	@Override
	public int compareTo(Person o) {
		return o.age - this.age;
	}
}
 
public static void main(String[] args) {
	TreeSet<Person> persons = new TreeSet<Person>();
	persons.add(new Person(43));
	persons.add(new Person(25));
	persons.add(new Person(111));
}
```

## Comparator
```java
class Person {
	int age;
 
	Person(int age) {
		this.age = age;
	}
}
 
class PersonAgeComparator implements Comparator<Person> {
	@Override
	public int compare(Person a, Person b) {
		return a.age - b.age;
	}
}
 
public static void main(String[] args) {
	TreeSet<Person> persons = new TreeSet<Person>(new PersonAgeComparator());
	persons.add(new Person(43));
	persons.add(new Person(25));
	persons.add(new Person(111));
}
```

# Threads
A *thread* is a unit of program execution that runs independently from other threads. Java programs can consist of multiple threads of execution that behave as if they were running on independent CPUs.

- `java.lang.Thread` is the Thread class representing a thread, which you can extend and then override its run() method. Afterwards, you call start().
- `java.lang.Runnable` is a functional interface (meaning only one method) which you can implement and then override run(). Afterwards, you can pass the object to a Thread instance and run start().
- The `synchronized` keyword is a modifier that can be used to write blocks of code, methods, or other resources to protect it in a multithreaded environment.
- `wait()` and `notify()` or `notifyAll()` methods of `java.lang.Object` can be used to suspend or wake up threads.

Besides the main thread, developers can instantiate their own when:

1. A custom class extends the `Thread` class 
1. A `Thread` is passed an implemented `Runnable` instance

Override the `run()` method in both cases to define the program to be run concurrently in the new thread, then call the Thread instance's `start()` method.

## Extend `Thread`
```java
public class CustomThread extends Thread {
    @Override
    public void run() {
        // Do something
    }

    public static void main(String[] args) {
        new CustomThread().start();
    }
}
```

## Implement `Runnable`
```java
public class CustomRunnable implements Runnable {
    @Override
    public void run() {
        // Do something
    }

    public static void main(String[] args) {
        new Thread(new CustomRunnable()).start();
    }
}
```

## Anonymous `Runnable` Class
```java
new Thread(new Runnable() {
        public void run() {
            // Do something
        }
    }).start();
```

## `Runnable` Lambda
```java
new Thread(
    () -> { /* Do something */ };
    ).start();
```

# JDBC API
Java Database Connectivity (JDBC) is an API for connecting to a RDBMS such as Oracle, PostgreSQL, or MySQL. As a collection of interfaces it requires a driver from the database vendor on the classpath. Once added, a `java.sql.Connection` is used to send SQL queries with `java.sql.Statement`, `java.sql.PreparedStatement`, or `java.sql.CallableStatement` objects, and retrieve result sets in `java.sql.ResultSet` objects.

```java
// Loading the driver may not be necessary, but it's good to specify
try {
    Class.forName("org.postgresql.Driver");
} catch (java.lang.ClassNotFoundException e) {
    System.out.println(e.getMessage());
}

// Pay attention to the url pattern
String url = "jdbc:postgresql://host:port/database";
String username = "databaseuser"
String password = "password"

try (
    // Be sure to close all connections after use
    Connection connection = DriverManager.getConnection(url, username, password);
    Statement statement = connection.createStatement();
){
    // executeUpdate() returns the number of rows affected for DML
    int rowCount = statement.executeUpdate("insert into pizza values (1, 'cheese')");

    // executeQuery() returns a ResultSet object for queries
    ResultSet pizzas = statement.executeQuery("select * from pizza");

    // Loop through ResultSet for each row returned
    while(pizzas.next()) {
        System.out.println(pizzas.getInt("id"));
        System.out.println(pizzas.getString("flavor"));
    }

} catch (SQLException ex) {
    
} 
```

## Statement
A Statement object sends queries and updates, as well as receive errors or ResultSets.

**Statement** is prone to SQL Injection attacks, especially if you use a raw string to write the query.

**PreparedStatement** is a precompiled SQL statement. It is best used for writing several similar queries in a loop, but will also as a side effect protect against SQL Injections
```java
PreparedStatement ps = myConnection.prepareStatement("UPDATE ANIMALS SET name=? WHERE id=?");
ps.setString(1, "Hippo");
ps.setInt(2, 7);
ps.executeQuery();
```
**CallableStatement** execute stored procedures and can return 1 or many ResultSets.
```java
CallableStatement cs = myConnection.prepareCall("{CALL BIRTHDAY_SP(?, ?)}");
cs.setInt(1, aid);
cs.setInt(2, yta);
cs.execute();
```


# SQL
## Terminology
**RDBMS** Relational Database Management System, relational referring to relational data (i.e. tables).

**Schema** Like packages/namespaces, groupings of tables expressing some database logical structure.

**SQL implementations** There is PostgreSQL is an Enterprise Database like Oracle, SQL Server, but there are others like MySQL/MariaSQL as well as non relational SQL databases (NoSQL).

**Candidate Key** A column that can uniquely identify a row (or entry) and thus is a potential candidate for a primary key.

**Composite Key** A primary key consisting of multiple columns.

**Primary Key** Unique (in that table), non-null candidate key.

**Foreign Key** A key that points to another primary key of a row (either in another table, or the same).

**Multiplicity** Refers to the relationship between linked tables. One-to-One (University, President), One-to-Many (University, Students), Many-to-Many (Students, Teachers). In 1:1, FKs will be within same table. 1:many, FKs will be in the other table. many:many, FKs will be in a junction/transition/join/lookup table.

**Referential Integrity** Enforcing data relationships, changes reflected between foreign keys. No orphans, all child rows must have their parent rows deleted as well.

**Domain Integrity** Column data is restricted to allowed range of allowed type.

**ERD** Entity-Relational Diagram

**Alias** The `AS` or `IS` keyword allows you to set a Table name or column name as a short variable.

**Normalization** Dividing data into separate tables to reduce redundancy and improve query speed

## Normal Forms
### Starting point (no normalization)
| SalesStaff |
| --- |
| EmployeeID |
| SalesPerson |
| SalesOffice |
| Age |
| DOB |
| Customer1 |
| Customer2 |
| Customer3 |

### 1st NF (Atomic values, No repeating Columns)
| SalesStaff |
| --- |
| EmployeeID |
| SalesPersonName |
| Age |
| DOB |
| SalesOfficeStreet |
| SalesOfficeCity |
| SalesOfficeState |
| SalesOfficeZip |

| Customer |
| --- |
| CustomerId |
| EmployeeId |
| CustomerName |

### 2nd NF (Remove Partial Dependencies)
| SalesStaff |
| --- |
| EmployeeID |
| SalesPersonName |
| Age |
| DOB |
| SalesOfficeID |

| SalesOffice |
| --- |
| SalesOfficeId |
| SalesOfficeStreet |
| SalesOfficeCity |
| SalesOfficeState |
| SalesOfficeZip |

| Customer |
| --- |
| CustomerId |
| EmployeeId |
| CustomerName |

### 3rd NF (Remove Transitive Dependencies)
| SalesStaff |
| --- |
| EmployeeID |
| SalesPersonName |
| DOB |
| SalesOfficeID |

| SalesOffice |
| --- |
| SalesOfficeId |
| SalesOfficeStreet |
| SalesOfficeZip |

| Customer |
| --- |
| CustomerId |
| EmployeeId |
| CustomerName |

## Sublanguages
**DCL** Data Control Language, setting user permissions (GRANT, REVOKE)

**DDL** Data Definition Language, working with database structure (CREATE, ALTER, TRUNCATE, DROP) EX:
```sql
CREATE TABLE (Schema)[TableName]
(Column definitions (Constraints))


ALTER TABLE [TableName]
ADD (Column) [Column definition]
ADD (Constraint clause)
DROP [column] [cascade]
DROP Constraint
ALTER COLUMN [definition]
```

**DML** Data Manipulation Language, working with the rows of data itself (INSERT, UPDATE, DELETE) EX:
```sql
INSERT INTO [TableName] [columns]
VALUES (data input)
SELECT (drop entire result set into table)
```

**DQL** Data Query Language, retrieving rows of data (SELECT). EX:
```sql
SELECT [columnList]
FROM [tableList]
WHERE [conditionList]
GROUP BY [columnList] // aggregate functions
HAVING [condition]  // aggregate functions
ORDER BY [columnList]
```

**TCL** Transaction Control Language, managing transactions (COMMIT, ROLLBACK, SAVEPOINT)
```sql
BEGIN;
SAVEPOINT this_point;
INSERT ...
INSERT ...
INSERT ... --Error here
ROLLBACK TO SAVEPOINT this_point; --Undo last 3 inserts
INSERT ...
COMMIT; --Only last insert will commit
```

## Joins
Combine rows from two tables based on some logical relationship between them (columns)
### Types
1. Inner Join, selects records with matching values from TableA and TableB
1. Left (Outer) Join, TableA primary, selects all records from A with matching values from B (non-matching values included as null)
1. Right (Outer) Join, TableB is primary, opposte of Left Join.
1. Cross Join, Cartesian join of two tables, if TableA has 5 rows, and TableB has 3 rows, the cross join will have 15 rows
1. Subquery is a query nested in the WHERE clause of a SELECT statement, in orde3r to further restrict the data returned. There are correlated and non-correlated. Correlated subqueries depend on the outer query to exist, meaning they cannot execute independently.

## Unions
1. UNION returns distinct rows present in either return set
1. UNION ALL returns all rows in both sets (including duplicates)
1. INTERSECT returns distinct rows present in both sets
1. MINUS returns all rows present in first set but not in second

## Functions
**Sequences** Generate numeric sequence, mostl for creating/managing primary keys.

**Views** Virtual table that displays the results of a SELECT statement, lets you reuse and store complex queries

**Indexes** Physical ordering of a column or group of columns, having unique indexes

**Aggregate Functions** (AVG, MIN, MAX, SUM, COUNT) perform an action on an entire column

**Scalar functions** (LOWER, UPPER) operate on individual entries

**Functions** Custom function with 0 or many input parameters, but 0 or 1 output. DML is not allowed.

**Stored Procedures** Custom function with 0 or many input parameters, but 0 or many output parameters. DML allowed.
