# PRODIGY_FS_01
:spring_version: current
:SpringApplication: https://docs.spring.io/spring-boot/docs/{spring_boot_version}/api/org/springframework/boot/SpringApplication.html
:toc:
:icons: font
:source-highlighter: prettify
:project_id: gs-authenticating-ldap

This guide walks you through the process creating an application and securing it with the https://projects.spring.io/spring-security/[Spring Security] LDAP module.

== What You Will Build

You will build a simple web application that is secured by Spring Security's embedded Java-based LDAP server. You will load the LDAP server with a data file that contains a set of users.

== What You Need

include::https://raw.githubusercontent.com/spring-guides/getting-started-macros/main/prereq_editor_jdk_buildtools.adoc[]

include::https://raw.githubusercontent.com/spring-guides/getting-started-macros/main/how_to_complete_this_guide.adoc[]

[[scratch]]
== Starting with Spring Initializr

NOTE: Because the point of this guide is to secure an unsecured web application, you will
first build an unsecured web application and, later in the guide, add more dependencies
for the Spring Security and LDAP features.

You can use this https://start.spring.io/#!type=gradle-project&language=java&platformVersion=3.1.0&packaging=jar&jvmVersion=17&groupId=com.example&artifactId=authenticating-ldap&name=authenticating-ldap&description=Demo%20project%20for%20Spring%20Boot&packageName=com.example.authenticating-ldap&dependencies=web[pre-initialized project] and click Generate to download a ZIP file. This project is configured to fit the examples in this tutorial.

To manually initialize the project:

. Navigate to https://start.spring.io.
This service pulls in all the dependencies you need for an application and does most of the setup for you.
. Choose either Gradle or Maven and the language you want to use. This guide assumes that you chose Java.
. Click *Dependencies* and select *Spring Web*.
. Click *Generate*.
. Download the resulting ZIP file, which is an archive of a web application that is configured with your choices.

NOTE: If your IDE has the Spring Initializr integration, you can complete this process from your IDE.

NOTE: You can also fork the project from Github and open it in your IDE or other editor.

[[initial]]
== Create a Simple Web Controller

In Spring, REST endpoints are Spring MVC controllers. The following Spring MVC controller
(from `src/main/java/com/example/authenticatingldap/HomeController.java`) handles a
`GET /` request by returning a simple message:

====
[source,java,tabsize=2]
----
include::initial/src/main/java/com/example/authenticatingldap/HomeController.java[]
----
====

The entire class is marked up with `@RestController` so that Spring MVC can autodetect the
controller (by using its built-in scanning features) and automatically configure the
necessary web routes.

`@RestController` also tells Spring MVC to write the text directly into the HTTP response
body, because there are no views. Instead, when you visit the page, you get a simple
message in the browser (because the focus of this guide is securing the page with LDAP).

== Build the Unsecured Web Application

Before you secure the web application, you should verify that it works. To do that, you
need to define some key beans, which you can do by creating an `Application` class. The
following listing (from
`src/main/java/com/example/authenticatingldap/AuthenticatingLdapApplication.java`) shows
that class:

====
[source,java,tabsize=2]
----
include::complete/src/main/java/com/example/authenticatingldap/AuthenticatingLdapApplication.java[]
----
====

include::https://raw.githubusercontent.com/spring-guides/getting-started-macros/main/spring-boot-application-new-path.adoc[]

include::https://raw.githubusercontent.com/spring-guides/getting-started-macros/main/build_an_executable_jar_subhead.adoc[]
include::https://raw.githubusercontent.com/spring-guides/getting-started-macros/main/build_an_executable_jar_with_both.adoc[]

If you open your browser and visit http://localhost:8080, you should see the following
plain text:

====
[source,text]
----
Welcome to the home page!
----
====

== Set up Spring Security

To configure Spring Security, you first need to add some extra dependencies to your build.

For a Gradle-based build, add the following dependencies to the `build.gradle` file:

====
[source,java,tabsize=2]
----
include::complete/build.gradle[tag=security,indent=0]
----
====

NOTE: Due to an artifact resolution issue with Gradle, *spring-tx* must be pulled in.
Otherwise, Gradle fetches an older one that doesn't work.

For a Maven-based build, add the following dependencies to the `pom.xml` file:

====
[source,xml]
----
include::complete/pom.xml[tag=security,indent=0]
----
====

These dependencies add Spring Security and UnboundId, an open source LDAP server. With
those dependencies in place, you can then use pure Java to configure your security policy,
as the following example (from
`src/main/java/com/example/authenticatingldap/WebSecurityConfig.java`) shows:

====
[source,java,tabsize=2]
----
include::complete/src/main/java/com/example/authenticatingldap/WebSecurityConfig.java[]
----
====


You also need an LDAP server. Spring Boot provides auto-configuration for an embedded
server written in pure Java, which is being used for this guide. The
`ldapAuthentication()` method configures things so that the user name at the login form is
plugged into `{0}` such that it searches `uid={0},ou=people,dc=springframework,dc=org` in
the LDAP server. Also, the `passwordCompare()` method configures the encoder and the name
of the password's attribute.

== Add Application Properties

Spring LDAP requires that three application properties be set in the `application.properties`` file:

====
[source,text]
----
spring.ldap.embedded.ldif=classpath:test-server.ldif
spring.ldap.embedded.base-dn=dc=springframework,dc=org
spring.ldap.embedded.port=8389
----
====

== Set up User Data

LDAP servers can use LDIF (LDAP Data Interchange Format) files to exchange user data. The
`spring.ldap.embedded.ldif` property inside `application.properties` lets Spring Boot pull
in an LDIF data file. This makes it easy to pre-load demonstration data. The following
listing (from `src/main/resources/test-server.ldif`) shows an LDIF file that works with this example:

====
[source,ldif]
----
include::complete/src/main/resources/test-server.ldif[]
----
====

NOTE: Using an LDIF file is not standard configuration for a production system. However,
it is useful for testing purposes or guides.

:module: secured web application

If you visit the site at http://localhost:8080, you should be redirected to a login page
provided by Spring Security.

Enter a user name of `ben` and a password of `benspassword`. You should see the following
message in your browser:

====
[source,text]
----
Welcome to the home page!
----
====

== Summary

Congratulations! You have written a web application and secured it with
https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/[Spring Security].
In this case, you used an
https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#ldap[LDAP-based user store].

== See Also

The following guide may also be helpful:

* https://spring.io/guides/gs/spring-boot/[Building an Application with Spring Boot]

include::https://raw.githubusercontent.com/spring-guides/getting-started-macros/main/footer.adoc[]
