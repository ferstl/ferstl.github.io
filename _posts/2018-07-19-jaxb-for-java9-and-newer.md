---
layout: post
title: Using JAXB in Java 9+
---

## TL;DR

If you need to use JAXB in Java 9 or higher versions, add this dependency to your project and you are fine:

```
<dependency>
  <groupId>org.glassfish.jaxb</groupId>
  <artifactId>jaxb-runtime</artifactId>
  <version>2.3.0</version>
</dependency>
```

## Slightly More Detailed Version

Starting with Java 9 some Java EE APIs that have been part of the Java platform for a long time have been deprecated and are likely to be removed in future Java releases. Java 9 encapsulates these APIs into modules which are marked as *deprecated for removal*. [JEP 261](http://openjdk.java.net/jeps/261) does specifically mention these modules for removal in future releases:

- `java.activation`
- `java.corba`
- `java.transaction`
- `java.xml.bind`
- `java.xml.ws`
- `java.xml.ws.annotation`

[JEP 320](http://openjdk.java.net/jeps/320), which is targeted for Java 11 (due in September 2018), proposes the effective removal of these modules.

This post focuses on the JAXB implementation which is currently used in many productive Java applications.

When migrating from Java 8 (or lower versions) to Java 9, such applications will neither compile nor run without taking further actions because the `java.xml.bind` module is not visible by default.

There are two possibilies to get such applications built and run with Java 9:

1. Explicitly add the `java.xml.bind` module at compile **and** at runtime
2. Add JAXB as external library to your application

Option 1) is simple. Just add `--add-modules java.xml.bind` as additional option to your `javac` **and** your `java` command line. However, as [JEP 320](http://openjdk.java.net/jeps/320) indicates, this will only work for Java 9 and 10. As of Java 11, the `java.xml.bind` module will most likely not be part of the Java platform anymore. So this solution will not be working quite soon.

Option 2) is adding a JAXB implementation as external library to your application. Up to version 10, the Java platform contains the JAXB reference implementation from the Glassfish project. Adding the reference implementation as external library will enable a smooth transition to Java versions beyond version 10.
Unfortunately, *"adding the JAXB reference implementation as external library"* is not as easy as it sounds. The Glassfish project published quite a few JAXB libraries, some with the groupId `com.sum.xml.bind` and some with the groupId `org.glassfish.jaxb`. So which is the right one? The answer is:

```
<dependency>
  <groupId>org.glassfish.jaxb</groupId>
  <artifactId>jaxb-runtime</artifactId>
  <version>2.3.0</version>
</dependency>
```

This library is the reference implementation of the JAXB API which is defined in `org.glassfish.jaxb:jaxb-core`. When you are using a build tool like Maven `jaxb-core` will be resolved transitively through `jaxb-runtime`.
Additionally, the `jaxb-runtime` library does also contain a Java service descriptor defining this reference implementation as JAXB provider.
The libraries with the groupId `com.sun.xml.bind` are only published for legacy purposes and should not be used.

So, adding `jaxb-runtime` as external library to your application will add the JAXB reference implementation as external component instead of using the build-in implementation to your application. If you have strict compatibility requirements, use version 2.2.10 as Java 8 replacement and version 2.3.0 as Java 9+ replacement.
