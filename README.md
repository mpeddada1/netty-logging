# netty-logging

To run this sample:

1) `java -version`:

```
openjdk version "11.0.15" 2022-04-19
OpenJDK Runtime Environment GraalVM CE 22.1.0 (build 11.0.15+10-jvmci-22.1-b06)
OpenJDK 64-Bit Server VM GraalVM CE 22.1.0 (build 11.0.15+10-jvmci-22.1-b06, mixed mode, sharing)
```

2) `mvn clean install`
3) `cd child-module`
4) `mvn package -Pnative`
5) Notice the warning:

```
Warning: class initialization of class io.grpc.netty.shaded.io.netty.util.internal.logging.Slf4JLoggerFactory failed with exception java.lang.NoClassDefFoundError: Could not initialize class io.grpc.netty.shaded.io.netty.util.internal.logging.Slf4JLoggerFactory. This class will be initialized at run time. Use the option --initialize-at-run-time=io.grpc.netty.shaded.io.netty.util.internal.logging.Slf4JLoggerFactory to explicitly request delayed initialization of this class.
Warning: class initialization of class io.grpc.netty.shaded.io.netty.util.internal.logging.Log4JLogger failed with exception java.lang.NoClassDefFoundError: org/apache/log4j/Priority. This class will be initialized at run time. Use the option --initialize-at-run-time=io.grpc.netty.shaded.io.netty.util.internal.logging.Log4JLogger to explicitly request delayed initialization of this class.
```

6) Uncomment the following dependency snippet in the pom.xml:

```
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-simple</artifactId>
  <version>2.0.0-alpha7</version>
  <scope>optional</scope>
</dependency>
```

7) Uncomment the `--initialize-at-build-time`  line in the pom.xml:

```
--initialize-at-build-time=org.slf4j.impl.SimpleLogger,org.slf4j.LoggerFactory
```

8) Notice the warning disappear.

The dependency is optional and only needed when we use the slf4j logging library
