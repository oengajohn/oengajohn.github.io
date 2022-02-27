# Getting Started with Arquillian

**Arquillian** is a platform that simplifies integration testing for Java middleware. It deals with all the plumbing of container management, deployment,
and framework initialization so that you can focus on the task of writing your tests-real
tests. Arquillian minimizes the burden on you-for the developer-by covering aspects
surrounding test execution; some of these aspects are as follows

* Managing the life cycle of the container (start/stop)
* Bundling the test class with the dependent classes and resources into a deployable
  archive
* Enhancing the test class (for example, resolving the `@Inject`,  `@EJB`, and `@Resource` injections)
* Deploying the archive to test applications (deploy/undeploy), and capturing results
  and failures.

Arquillian also has extensions that enhance its features, for example, allowing it to
perform UI tests or some nonfunctional tests.

## Setting up an Arquillian Project with Maven

Depending on the type of archetype you used for generation, you might have a different
folder structure in your project; this is not an issue. What is really important is that you
provide the following structure under your src folder:

* `main/java/`: Place all application Java source files here (under the Java package)
* `main/resources/`: Place all application configuration files here
* `test/java/`: Place all test Java source files here (under the Java package)
* `test/resources/`: Place all test configuration files here (for example, `persistence.xml`)

So by now, we will be working under `test/java` , which is where we will place our first Arquillian test class.

## Writing an Arquillian test

* If you have been working with [JUnit](http://www.junit.org), you will find a similar Arquillian test with some extra spice on it.

## Configuring the `pom.xml` file

The first thing that is necessary to include in order to run an Arquillian test is the junit dependency, which is required to run our unit tests:

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.8.2</version>
</dependency>
```

Next we will use the Arquillian **Bill of Materials (BOM)** in order to import versions of all Arquillian-related dependencies:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.jboss.arquillian</groupId>
            <artifactId>arquillian-bom</artifactId>
            <version>1.7.0.Alpha10</version>
            <scope>import</scope>
            <type>pom</type>
        </dependency>
    </dependencies>
</dependencyManagement>
```

We're using Arquillian with JUnit so we need to include the appropriate dependency in the dependencies tag:

```xml
<dependency>
    <groupId>org.jboss.arquillian.junit5</groupId>
    <artifactId>arquillian-junit5-container</artifactId>
</dependency>
```

After being done with the basic dependencies, we now have to specify the container against which the tests will be run. Container adapters are available for the more important
Java EE Application Servers (WildFly, Glassfish, WebLogic, and WebSphere), as well as for some servlet containers such as Tomcat or Jetty. Here, we want to use WildFly so we will use an appropriate container adapter. However, we have a few possible choices.
Container adapters can be divided into three basic groups:

* **Embedded**: This is the mode in which a container is run on the same JVM instance the tests are running. Often, a container run in this mode is not an original one, but
packed to a single JAR limited version.
* **Managed**: In this mode, the real application server is run on a separate JVM. As the name implies, it's possible to manage the state of the container, run it, stop it, and so
on. By default, when you run the test, the server is started, tests are run against it, and then it is stopped. However, it is possible to configure Arquillian to run tests on the
already running instance.
* **Remote**: In this mode, we just connect to some existing server instance and run tests against it.

The most universal choice to run tests is the managed container. Tests are run on the real server, same as on the production environment, and additionally, it is possible to manage its state, allowing for some more advanced tests such as testing features related to high-availability or communication between two applications that run on different instances.Now, we need to add the appropriate container adapter to our `pom.xml` file. To do this, we
will create a Maven profile under the profiles section of the `pom.xml` file:

```xml
<profile>
    <id>arq-wildfly-managed</id>
    <dependencies>
        <dependency>
            <groupId>org.wildfly.arquillian</groupId>
            <artifactId>wildfly-arquillian-container-managed</artifactId>
            <version>5.0.0.Alpha1</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</profile>
```

There might be situations in which you'd like to run tests against different application servers. It's possible to just define a few Maven profiles and run tests a few times, each
time activating other profiles. Keep in mind that some application servers don't provide all types of the adapters.

Our Arquillian tests use a protocol to communicate with the micro deployment on the application server. If we don't specify the protocol, the container will choose the default one. In order to specify it manually, we will need to add the org.jboss.arquillian.protocol dependency (named so as it's
compatible with Servlet 3.0 specifications):

```xml
<dependency>
    <groupId>org.jboss.arquillian.protocol</groupId>
    <artifactId>arquillian-protocol-servlet</artifactId>
    <scope>test</scope>
</dependency>
```

By doing the above settings our `pom.xml` file should look almost similar to below:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>io.jotech</groupId>
    <artifactId>arquillian</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>arquillian</name>
    <packaging>war</packaging>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.target>11</maven.compiler.target>
        <maven.compiler.source>11</maven.compiler.source>

    </properties>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.jboss.arquillian</groupId>
                <artifactId>arquillian-bom</artifactId>
                <version>1.7.0.Alpha10</version>
                <scope>import</scope>
                <type>pom</type>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0.1</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.8.2</version>
        </dependency>
        <dependency>
            <groupId>org.jboss.arquillian.junit5</groupId>
            <artifactId>arquillian-junit5-container</artifactId>
        </dependency>
        <dependency>
            <groupId>org.jboss.arquillian.protocol</groupId>
            <artifactId>arquillian-protocol-servlet</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.3.2</version>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.22.2</version>
            </plugin>

        </plugins>
    </build>
    <profiles>
        <profile>
            <id>arq-wildfly-managed</id>
            <dependencies>
                <dependency>
                    <groupId>org.wildfly.arquillian</groupId>
                    <artifactId>wildfly-arquillian-container-managed</artifactId>
                    <version>5.0.0.Alpha1</version>
                    <scope>test</scope>
                </dependency>
            </dependencies>
        </profile>
    </profiles>
</project>
```

## Writing your first Arquillian test

Once the configuration is complete, we will finally code our test. So, create a Java class named `CalculatorTest` under the package`io.jotech.arquillian`. The first thing that you will add to this class is the following annotation that tells JUnit to use `ArquillianExtension` as the test controller:

```java
@ExtendWith(ArquillianExtension.class)
class CalculatorTest {

}
```

Arquillian then looks for a static method with the `@Deployment` annotation; it creates a micro deployment including all the specified classes and resources (instead of deploying the whole application), allowing to test only part of the system:

```java
    @Deployment
    public static Archive<?> createTestArchive() {
        return ShrinkWrap.create(WebArchive.class,"test.war")
                .addPackage(Calculator.class.getPackage())
                .addAsWebInfResource(EmptyAsset.INSTANCE, "beans.xml");
    }
```

The fluent API provided by the [ShrinkWrap](http://www.jboss.org/shrinkwrap) project makes this technique possible using the create method, which accepts the type of deployment unit (***WebArchive***) as the argument and all the resources are included in this archive. In our case, instead of including all the single classes, we use the `addPackage`
utility method that adds all the classes that are contained in a class package (for example,
by adding the `Calculator.class.getPackage()` method, we will include all the classes that
are in the same package as the `Calculator` class). And,
of course, we also have to add the empty beans.xml file in order to enable the CDI.

Finally, we inject the service we would like to test and add one test method:

```java
    @Inject
    private Calculator calculator;

    @Test
    public void add() {
        //given
        var a = 12;
        var b = 14;
        var numbers = new int[]{};
        //when
        var actual = calculator.add(a,b,numbers);
        //then
        var expected = 26;
        assertEquals(expected,actual);
    }
```

Our first test case is now ready. We will just need to add an Arquillian configuration file named `arquillian.xml` in our project, under `src/test/resources`:

```xml
    <arquillian xmlns="http://jboss.org/schema/arquillian" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xsi:schemaLocation="http://jboss.org/schema/arquillian
        http://jboss.org/schema/arquillian/arquillian_1_0.xsd">
        <container qualifier="widlfly-managed">
        </container>
    </arquillian>
```

You have to configure the container adapter. In this example, we assume that you have set
the `JBOSS_HOME` environment variable to the WildFly main directory. In this case, no more configurations are required. However, if you want to run something non-standard, for
example, connect to a remote container with altered management ports, then this file is the appropriate place to modify this. When you don't specify `JBOSS_HOME`, you can set the
WildFly location using property as follows:

```xml
<container qualifier="widlfly-managed" default="true">
        <configuration>
            <property name="jbossHome">D:\workspace\servers\wildfly-24.0.1.Final</property>
</configuration>
</container>
```

However, this method may be hard to maintain when more than one person is working on the project. In order to avoid problems, you can use the system property resolution, for
instance, `${jbossHome}`.
If you configure the remote container, the configuration would look just like this:

```xml
 <container qualifier="widlfly-remote">
    <configuration>
        <property name="managementAddress">127.0.0.1</property>
        <property name="managementPort">9990</property>
        <property name="username">admin</property>
        <property name="password">admin</property>
    </configuration>
</container>
```

## Running Arquillian Calculator Test

It's possible to run Arquillian tests both from Maven and your IDE. You have to remember that we declared the container adapter in the Maven profile, so in order to run the full build, you have to run the following command line:

```console
mvn clean test -Parq-wildfly-managed
```

If you want to use only one container in your test, then a good idea would be to set the default Maven profile, by adding the following lines to it in the `pom.xml` file:

```xml
<activation>
    <activeByDefault>true</activeByDefault>
</activation>
```

## ShrinkWrap Resolver

In almost every Arquillian test, you will probably use ShrinkWrap to create micro deployments. After working with it for a bit, you will probably notice some shortcomings. You might be wondering what happens when you've got a test that relies on some external library; do you need to add all packages from that library? The answer is no. **ShrinkWrap
Resolver** offers integration with Maven and basic Gradle support is also available. You can just write in your test what dependency you'd like to include in the archive and it will be deployed with the micro deployment.

Let's look at the basic example of the ShrinkWrap Resolver Maven integration:
`Maven.resolver().resolve("G:A:V").withTransitivity().asFile();`

The preceding line means that we want to resolve an artifact with the given group ID, artifact ID, and version (Maven coordinates in canonical form) from Maven's central
repository with all its dependencies, and convert it to a list of files.

However, with this example, you have to maintain the artifact version both in the test code and build file. You can improve this! Just import some dependencies data from your
pom.xml file, so that ShrinkWrap Resolver resolves artifacts of the same versions the main project is using:
`Maven.resolver().loadPomFromFile("/path/to/pom.xml").
resolve("G:A").withTransitivity().asFile();`
<Note type="tip">
Arquillian-bom does not include this ShrinkWrap Resolver version. <br />
However, it is possible to override the BOM imported versions with another BOM. It's pretty easy; just insert the more important BOM first:
</Note>

```xml

    <dependencies>
        <!-- shrinkwrap resolvers import must be before arquillian bom! -->
        <dependency>
            <groupId>org.jboss.shrinkwrap.resolver</groupId>
            <artifactId>shrinkwrap-resolver-bom</artifactId>
            <version>3.1.4</version>
            <scope>import</scope>
            <type>pom</type>
        </dependency>
        <dependency>
            <groupId>org.jboss.arquillian</groupId>
            <artifactId>arquillian-bom</artifactId>
            <version>1.7.0.Alpha10</version>
            <scope>import</scope>
            <type>pom</type>
        </dependency>
    </dependencies>

```

And then include the following dependency:

```xml
<!-- shrinkWrap resolvers -->
<dependency>
    <groupId>org.jboss.shrinkwrap.resolver</groupId>
    <artifactId>shrinkwrap-resolver-depchain</artifactId>
    <scope>test</scope>
    <type>pom</type>
</dependency>
```

Let's say you're testing your project using JUnit and some fancy assertion library. `AssertJ` is a fluent assertions library that allows you to write your project in a more human-readable form:
`assertThat(frodo.getName()).isEqualTo("Frodo");`

Using such a library in every test means you have to include it in every micro deployment.
There is another thing you will always need: the beans.xml file. So let's create some
utility classes:

```java

import java.io.File;
import org.jboss.shrinkwrap.api.ShrinkWrap;
import org.jboss.shrinkwrap.api.asset.EmptyAsset;
import org.jboss.shrinkwrap.api.spec.WebArchive;
import org.jboss.shrinkwrap.resolver.api.maven.Maven;
import org.jboss.shrinkwrap.resolver.api.maven.archive.importer.MavenImporter;

public class ArquillianWarUtils {
    private static final String ASSERTJ_COORDINATE =
            "org.assertj:assertj-core";
    private static final File[] ASSERTJ_ARTIFACT = Maven.resolver()
            .loadPomFromFile("pom.xml")
            .resolve(ASSERTJ_COORDINATE)
            .withTransitivity().asFile();

    public static WebArchive getBasicWebArchive() {
        return  ShrinkWrap.create(WebArchive.class, "test.war")
                .addAsLibraries(ASSERTJ_ARTIFACT)
                .addAsWebInfResource(EmptyAsset.INSTANCE, "beans.xml");
    }

    public static WebArchive importBuildOutput() {
        return ShrinkWrap.create(MavenImporter.class).loadPomFromFile("pom.xml")
                .importBuildOutput()
                .as(WebArchive.class);
    }
}
```
