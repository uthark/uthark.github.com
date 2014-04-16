---
layout: post
title: "Using AssertJ"
date: 2014-04-15 23:37:49 -0400
comments: true
categories: 
    - development
    - article
    - java
    - unit testing
---

[AssertJ] is a library which provides fluent strongly-typed assertions to use in unit tests.

Example of assertions written with `AssertJ`:

``` java 
import io.github.uthark.blog.assertj.Assertions.assertThat;

// ... within @Test
User result = userDao.findByLogin("username");
assertThat(result).
    isNotNull().
    isActive().
    hasLogin("username");

```

As you can see assertions look much more readable. 
The integration of assertj into Maven project is very easy:

1. Update pom.xml in parent module

``` xml pom.xml changes in parent module.
<properties>
    <assertj-core.version>1.6.0</assertj-core.version>
    <assertj-assertions-generator-maven-plugin.version>1.2.0</assertj-assertions-generator-maven-plugin.version>
</properties>

<dependencyManagement>
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <version>${assertj-core.version}</version>
        <scope>test</scope>
    </dependency>
</dependencyManagement>

<pluginManagement>
    <plugin>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-assertions-generator-maven-plugin</artifactId>
        <version>${assertj-assertions-generator-maven-plugin.version}</version>
        <executions>
            <execution>
                <id>generate-assertions</id>
                <phase>generate-test-sources</phase>
                <goals>
                    <goal>generate-assertions</goal>
                </goals>
            </execution>
        </executions>
    </plugin>

</pluginManagement>
```

2. Update pom.xml in child module
``` xml Changes in child module.
<dependencies>
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
    </dependency>
</dependencies>
<build>
    <plugins>
        <plugin>
            <groupId>org.assertj</groupId>
            <artifactId>assertj-assertions-generator-maven-plugin</artifactId>
            <configuration>
                <!-- you can define collection packages: -->
                <packages>
                    <param>io.github.uthark.blog.assertj</param>
                </packages>

                <!-- Or specify classes one by one. -->
                <classes>
                    <param>io.github.uthark.blog.assertj.User</param>
                </classes>
            </configuration>
        </plugin>
    </plugins>
</build>
```

After this changes it is possible to generate assertions for the classes you want test:

``` sh
mvn assertj:generate-assertions
```

By default assertj will generate assertion classes in `target/generated-test-sources/assertj-assertions/`, it is possible to override this behaviour by passing configuration element `targetDir` to plugin.

Additonal documentation for Maven plugin can be [found here](http://joel-costigliola.github.io/assertj/assertj-assertions-generator-maven-plugin.html)

Happy hacking!

[AssertJ]: https://github.com/joel-costigliola/assertj-core
