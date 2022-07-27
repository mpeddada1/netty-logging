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

8) Notice the following warning disappear:

```
Warning: class initialization of class io.grpc.netty.shaded.io.netty.util.internal.logging.Slf4JLoggerFactory failed with exception java.lang.NoClassDefFoundError: Could not initialize class io.grpc.netty.shaded.io.netty.util.internal.logging.Slf4JLoggerFactory. This class will be initialized at run time. Use the option --initialize-at-run-time=io.grpc.netty.shaded.io.netty.util.internal.logging.Slf4JLoggerFactory to explicitly request delayed initialization of this class.
```

**Note**: This dependency in this sample is only used for the purpose of addressing the warning. As described [here](https://github.com/netty/netty/blob/c08beb543a6b8db0c7f3cee225042361b4025e4c/common/src/main/java/io/netty/util/internal/logging/InternalLoggerFactory.java#L36), `InternalLoggerFactory` in the netty library first chooses `Slf4jLoggerFactory` as the default logger factory and if slf4j is not present then it chooses `Log4JLoggerFactory` and so on. This results in `io.grpc.netty.shaded.io.netty.util.internal.logging.Slf4JLoggerFactory` and `io.grpc.netty.shaded.io.netty.util.internal.logging.Log4JLogger` being initialized and can sometimes lead to the following warning by graal during native image compilation (when slf4j is not present on the classpath):

```
Warning: class initialization of class io.grpc.netty.shaded.io.netty.util.internal.logging.Slf4JLoggerFactory failed with exception java.lang.NoClassDefFoundError: Could not initialize class io.grpc.netty.shaded.io.netty.util.internal.logging.Slf4JLoggerFactory. This class will be initialized at run time because option --allow-incomplete-classpath is used for image building. Use the option --initialize-at-run-time=io.grpc.netty.shaded.io.netty.util.internal.logging.Slf4JLoggerFactory to explicitly request delayed initialization of this class.
```


To avoid this warning, we initially followed the instructions to initialize these classes at runtime. While this behaves fine when there is no slf4j or log4j dependency on the classpath, this breaks behavior when these logging dependencies are actually added. 

This warning can be misleading especially in a situation where a dependency is optional. Is there a workaround to not have the build result in this warning?
