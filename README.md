# Java Debugging in Visual Studio Code 

- [Gradle version](https://github.com/kasecato/vscode-javadebug-sample)

## Getting Started

### Install Extension

* Microsoft, Java Extension Pack, https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack

### Setting the JDK

* AdoptOpenJDK, https://adoptopenjdk.net/
* Amazon, Corretto 8, https://docs.aws.amazon.com/corretto/latest/corretto-8-ug/downloads-list.html
* Amazon, Corretto 11, https://docs.aws.amazon.com/corretto/latest/corretto-11-ug/downloads-list.html
* Azule Systems, Zule OpenJDK, https://www.azul.com/downloads/
* OpenJDK, https://jdk.java.net/
* Red Hat, OpenJDK, https://developers.redhat.com/products/openjdk/download/

The path to the Java Development Kit is searched in the following order:

* the `java.home` setting in VS Code settings (workspace then user settings)
* the `JDK_HOME` environment variable
* the `JAVA_HOME` environment variable
* on the current system path

### Maven Configurations

* pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>hello-world</artifactId>
    <version>0.1-SNAPSHOT</version>
    <packaging>war</packaging>
    <name>Jetty HelloWorld WebApp</name>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.8</java.version>
        <maven.compiler.target>${maven.compiler.source}</maven.compiler.target>
        <jettyVersion>9.4.27.v20200227</jettyVersion>
        <junit.jupiter.version>5.6.1</junit.jupiter.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>${junit.jupiter.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-params</artifactId>
            <version>${junit.jupiter.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <version>${junit.jupiter.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.eclipse.jetty</groupId>
                <artifactId>jetty-maven-plugin</artifactId>
                <version>${jettyVersion}</version>
            </plugin>
            <plugin>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.22.2</version>
            </plugin>
        </plugins>
    </build>
</project>
```

### Build Configurations

* .vscode/tasks.json
```json
{
  "version": "2.0.0",
  "echoCommand": true,
  "command": "./mvnw",
  "presentation": {
    "echo": true,
    "reveal": "always",
    "focus": false,
    "panel": "shared"
  },
  "tasks": [
    {
      "label": "clean",
      "args": ["clean"],
      "type": "shell"
    },
    {
      "label": "validate",
      "args": ["validate"],
      "type": "shell"
    },
    {
      "label": "compile",
      "args": ["compile"],
      "type": "shell",
      "group": {
        "kind": "build",
        "isDefault": true
      }
    },
    {
      "label": "test",
      "args": ["test"],
      "type": "shell",
      "group": {
        "kind": "test",
        "isDefault": true
      }
    },
    {
      "label": "package",
      "args": ["package"],
      "type": "shell"
    },
    {
      "label": "verify",
      "args": ["verify"],
      "type": "shell"
    },
    {
      "label": "install",
      "args": ["install"],
      "type": "shell"
    },
    {
      "label": "jetty:run",
      "args": ["jetty:run"],
      "type": "shell"
    }
  ]
}
```

### Java Debugger Configurations

* .vscode/launch.json
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "java",
            "name": "Debug (Attach)",
            "request": "attach",
            "hostName": "localhost",
            "port": 8000
        }
    ]
}
```

### Java programming

* src/main/org/example/HelloServlet.java
```java
package org.example;

import java.io.IOException;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class HelloServlet extends HttpServlet {

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html");
        response.setStatus(HttpServletResponse.SC_OK);
        response.getWriter().println("<h1>Hello Servlet</h1>");
        response.getWriter().println("session=" + request.getSession(true).getId());
    }

}
```

* src/webapp/WEB-INF/web.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app
   xmlns="http://xmlns.jcp.org/xml/ns/javaee"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
   metadata-complete="false"
   version="3.1">

  <servlet>
    <servlet-name>Hello</servlet-name>
    <servlet-class>org.example.HelloServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>Hello</servlet-name>
    <url-pattern>/hello/*</url-pattern>
  </servlet-mapping>

</web-app>
```


* src/main/java/com/vscode/demo/MainTest.java
```java
package com.vscode.demo;

import java.util.Arrays;
import java.util.List;
import java.util.Objects;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Tag;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertEquals;

@Tag("main")
class MainTest {

    @Test
    @DisplayName("VS Code JUnit 5 test")
    void testMain() {

        // arrange
        final List<String> list = Arrays.asList("React", "Angular", "Vue");

        // act
        final String actual = list.stream()
            .filter(x -> Objects.equals(x, "React"))
            .findFirst()
            .orElse(null);

        // assert
        assertEquals("React", actual, () -> "Main Succeed");
    }

}
```

### Debugging

1. Set debugger
```bash
export MAVEN_OPTS="-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=8000 -Xnoagent -Djava.compiler=NONE"
```

2. Run jetty
```bash
./mvnw jetty:run
```

3. Press `F5`

4. Open web
```bash
open http://localhost:8080/hello
```

### Testing

1. Open Command Pallete `cmd+shift+p` (macOS) or `ctrl+shift+p` (Windonws/Linux)
1. `Tasks Run Test Task`
1. `test`

# References

1. Visual Studio Code, Java in VS Code, https://code.visualstudio.com/docs/languages/java
1. GitHub, Language Support for Java(TM) by Red Hat, https://github.com/redhat-developer/vscode-java
1. Visual Studio Code, Integrate with External Tools via Tasks, Variable substitution, https://code.visualstudio.com/Docs/editor/tasks#_variable-substitution
1. Gradle, Chapter 47. The Java Plugin, https://docs.gradle.org/current/userguide/java_plugin.html
1. GitHub, JUnit 5 Samples, https://github.com/junit-team/junit5-samples
1. Gradle, Gradle 4.6 Release Notes - JUnit 5 support, https://docs.gradle.org/4.6/release-notes.html#junit-5-support
1. Maven, Configuring the Jetty Maven Plugin, https://www.eclipse.org/jetty/documentation/9.4.x/jetty-maven-plugin.html
