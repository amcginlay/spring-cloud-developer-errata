# Spring Cloud Developer Course Errata

## Logistics and Introduction

### Introduction of Instructors

- Alan McGinlay [amcginlay@pivotal.io](mailto:amcginlay@pivotal.io)
- Bill Kable [bkable@pivotal.io](mailto:bkable@pivotal.io)

### Student Intros

-   How many of you have been working on Java programming more than
    1 year, 3 years, 5 years?
-   How many of you have worked with Spring, Spring Boot?
-   How many of you have worked with, or are currently working with PAAS
    platforms such as Cloud Foundry or Kubernetes?
    If you are currently working with one now, which one?
-   What are your objectives for attending this class?

### Pairing

- Pivotal encourages pairing.
- Share knowledge with your pair.
- Share knowledge with your team.
- Communicate often, language barriers.

## Workshop Agenda

- Class starts at 9AM and ends at 5PM
- Two 15 minutes breaks, one in the morning and one in the afternoon
- Lunch is 1 hour around noon
- [Course Site](https://courses.education.pivotal.io/c/349802743/)

## Review Prequisites

-   [Review lab setup guide](https://courses.education.pivotal.io/c/349802743/spring-cloud-developer/setup/index.html)
-   Review all students have necessary prereqs.
-   There are places in the course where explicit *nix commands are
    given.
    We will call out alternatives in the errata.
-   Optionally setup a remote for your local code to push, if you want
    to save outside of class.

## Intro

### Builds

-   You are not building new functionality in this course.
    The only code you will write will be to add *platform-services*.

-   Builds are generally not required in this class.
    For most labs, running Gradle `bootRun` is sufficient.

-   Tests are not fully implemented on all the labs,
    and some will fail if you attempt to run them or build the project.
    If you attempt to run `./gradlew build` and it fails,
    run `./gradlew assemble` instead.

### Application Continuum (Lab)

-   Are you familiar with Multi-Module builds in either Gradle or Maven?
    *Pal Tracker Distributed* project is a gradle multi-module build,
    leveraging include files to keep level of redundancy to a minimum.

-   Instead of using "curl" as following:

    ```bash
    curl -i -XPOST -H"Content-Type: application/json" localhost:8084/time-entries/ -d"{\"projectId\": 1, \"userId\": 1, \"date\": \"2015-05-17\", \"hours\": 6}"
    ```
    You can use "httpie" as following:

    ```bash
    http post localhost:8084/time-entries projectId=1 userId=1 date=2015-05-17  hours=6
    ```

    You may also use the supplied Postman collection downloaded with the
    this errata site.
    It is set for localhost, and there are different folders by lab.
    The *Timesheet Entry Flow* executes the flow and asserts the
    appropriate response codes.

-   If you are using Ultimate Edition IntelliJ or STS, you might as well take
    advantage of "Spring Boot Dashboard", which makes running multiple applications
    easier to deal with.

-   If you are a Windows user and decided to use "cmd" terminal window for
    running apps, you might consider to use [ConEMU](https://conemu.github.io/) instead

### Spring Cloud Dependencies

-   Look over the interfaces in `spring-cloud-commons` jar:
    - Discovery
    - Service Registry
    - Load Balancer

-   Maven Central used for the labs, but Enterprise customers will
    leverage their own internal Maven Repository?

-   What Maven repo product is your organization using?

-   Are you running Docker containers?
    If so, what repo do you consume your Container images?

## Service Discovery

### Eureka Service Registry (Lab)

-   You are standing up Eureka Server as a Spring Boot application from
    scratch.
    So you will have to create `src/main/java` directory first before
    creating `io.pivotal.pal.tracker.eurekaserver.EurekaServerApp.java`

    You will have to create `src/main/resources` directory first before
    creating `application.yml` underneath it.
    This is the only place we use YAML based application properties in
    the course.
    YAML maps are a better rendered format for Eureka replication
    configurations.

-   Unlike IntelliJ, Eclipse/STS will not display the
    mult-module project
    in a "easy to understand" hierarchical fashion.
    Instead, it will display all projects in "flat" directory style.
    Just choose correct directory.

-   If you experience the following problem when running Eureka server,
    please use JDK 8 instead of JDK 9+.

    ```bash
    Exception in thread "main" java.lang.NoClassDefFoundError: javax/xml/bind/JAXBException
    ```

    Or add the following dependency to the build.gradle.

    ```groovy
    compile "javax.xml.bind:jaxb-api:2.3.0"
    ```

### Service Discovery Client (Lab)

-   Note that there are three different ways to create discovery client
    -   Using `ServiceLocator` interface and `EurekaServiceLocator`
        implementation class (lab)
    -   Using `DiscoveryClient` (challenge lab exercise) -> Make sure
        you use the Spring Cloud `DiscoveryClient`, NOT the Netflix OSS
        `DiscoveryClient`!
    -   Using `@LoadBalanced` annotation with `RestTemplate` (in the
        subsequent lab)

### Eureka REST Client Lab

-   Feel free to use the Postman *Eureka REST Endpoints* requests in
    place of `curl` or `httpie`.

### Eureka Health Check Lab

-   You may notice during the recovery of the Timesheets Server when
    The actuator health indicator reports "UP", but Eureka has not yet
    recovered from renewal.
-   During this time Eureka Remote Status of Timesheets Server will
    still report it as "DOWN", which will roll up the the Timesheets
    Server aggregate status.
    This will clear following the next Timesheets Server renewal.

### Ribbon Load Balancing

-   Refer back to the Discovery Client lab challenge, where you use
    the `DiscoveryClient`?:
    How would you use the `LoadBalancerClient` API?

-   Consider using the `scripts/DemonstrateLoadBalancing.jmx` Jmeter
    test to exercise your endpoint.

-   Be mindful of starting order of your apps:
    1. Eureka
    1. Registration Servers
    1. Timesheet App

-   Challenge for the Ribbon Load Balancing Labs:
    Run 3 or 5 instances of `registration-server` with various
    weightings, ramp up load rate to 5-10 requests per second and see
    if the algorithm holds over several minutes of time.

-   If you are running Client Load Balancing, Discovery in PCF, how does
    this work?
    Instructor show demo:
    - Service Registry Tile (Eureka)
    - Pivotal Spring Cloud Services Starters (Connector and Client)
    - Zero client config through VCAP
    - Container to Container networking

## Fault Tolerance

### Hystrix Circuit Breaker

-   Note you no longer have Eureka Server and Service Discovery.
    They are not necessary for using the Hystrix client.

-   You are standing up Hystrix Dashboard as a Spring Boot application
    from scratch.
    So you will have to create `src/main/java` directory first before
    creating
    `io.pivotal.pal.tracker.hystrixdashboard.HystrixDashboardApp.java`

-   Recommend to use the supplied `scripts/DemonstrateHystrix.jmx`
    jmeter test plan to better demonstrate the circuit breaker.

-   Idiosyncracies of Hystrix Dashboard:
    -   You may get an error "Cannot connect to stream" when you first
        access the timesheets server hystrix stream URL through the
        dashboard, if you have not yet generated load.
        Generate some load, and refresh the Hystrix Dashboard.
    -   Beware the default sort order of Hystrix protected circuits

### Hystrix Isolation Strategies (Lab)

-   In the JMeter, select HTTP Request, Advanced, Timeouts, and
    set Connect and Response timeout to 1000 and 10000

-   There is an comment error in the “application.properties” file
    of the "timesheets-server".
    Leave the value at 2000ms (2 seconds).

    ```properties
    # requests that take more than 5 seconds will “fail fast”:
    hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=2000
    ```

-   When instructed to review the instrumentation code, look at
    following:

    ```java
    applications/registration-server/src/main/io/pivotal/pal/tracker/registration/InstrumentedLatencyConfig.java
    ```

-   Make sure to refresh you hystrix dashboard after switching the
    timesheets server to `SEMAPHORE` mode and its restart.
    Otherwise you will see the thread pool from previous default config
    but no stats.

-   Why are you overriding the `RestOperations` at the `TimesheetsApp`?
    The `RestTemplateBuilder` is a Spring Boot dependency, ideally it
    should not live in the component, but at the application.
    This may raise design question of having a `rest-support` component,
    but beyond the scope of this course.

### Hystrix Stats Aggregation (Lab - Optional)

- The command to start RabbitMQ is "rabbitmq-server".

### Hystrix Demo

If you are interested in experimenting with the hystrix demo code, you
can get it [here](./hystrix-demo)

## Config Server and Spring Cloud Bus

-   See the [Spring Cloud Bus documentation](http://cloud.spring.io/spring-cloud-static/spring-cloud-bus/2.0.0.RELEASE/single/spring-cloud-bus.html)
    for how to use and/or customize.

-   See the [sample code](./spring-cloud-bus-demo)
    to demonstrate how to use custom `RemoteApplicationEvent` with Spring
    Cloud Bus and its Event listeners.

### Vault Backend

-   Verison 0.10.x introduces breaking changes.
    Install an
    [older compatible version](https://releases.hashicorp.com/vault/0.9.6/)
    Choose the correct version for your OS, unzip the binary, and copy
    onto the path (e.g. /usr/local/bin for *nix, or C:/Program Files
    on Windows).

### Distributed Updates (lab - Optional)

-   The RabbitMQ management and trace plugins can be installed as
    following on *nix based systems:

    ```bash
    sudo rabbitmq-plugins enable rabbitmq_management
    sudo rabbitmq-plugins enable rabbitmq_tracing
    ```

    Or the following on Windows:

    ```powershell
    rabbitmq-plugins enable rabbitmq_management
    rabbitmq-plugins enable rabbitmq_tracing
    ```

## Additional Topics

### Distributed Trace - Zipkin (Lab - Optional)

-   The document says the following:

    ```bash
    git checkout -b my-zipkin-start zipkin-start
    ```

    In the original lab version you downloaded in course version 1.0.24,
    there was not a "zipkin-start" tag provided.
    The latest updated course has it.
    Make sure you download the
    [latest lab code](https://courses.education.pivotal.io/c/349802635/codebases/spring-cloud-developer-code.zip).

    You will also need to download and run "zipkin.jar" using the
    following instruction (if you were using original lab version)

    ```bash
    curl -sSL https://zipkin.io/quickstart.sh | bash -s
    java -jar zipkin.jar
    ```