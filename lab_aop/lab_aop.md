# Using Aspect Oriented Programming in Micronaut

## Introduction
In this lab you will learn how Aspect Oriented Programming (AOP) allows you to define functionality to address cross cutting concerns in your application.

Estimated Lab Time: 10 minutes

### Objectives

In this lab you will:
* How to write a method interceptor
* How to apply the interceptor to your code

### Prerequisites
- Access to your project instance

## Define a MethodInterceptor

It is often a requirement to define cross-cutting concerns (logging, security, tracing, validation etc.) that apply to many methods in an application.

This is where Micronaut's support for [Aspect Oriented Programming (AOP)](https://docs.micronaut.io/latest/guide/index.html#aop) comes to the fore. There are several types of "advice" you can define in a Micronaut application. We are going to look at "Around" advice which is the most simple by defining a [MethodInterceptor](https://docs.micronaut.io/latest/api/io/micronaut/aop/MethodInterceptor.html) that logs the result of a method execution.

Firsts define a class in a file called `src/main/java/example/micronaut/LoggingInterceptor.java` with the following contents:

    <copy>
    package example.micronaut;

    import io.micronaut.aop.MethodInterceptor;
    import io.micronaut.aop.MethodInvocationContext;
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;

    import javax.inject.Singleton;

    @Singleton
    public class LoggingInterceptor implements MethodInterceptor<Object, Object> {
        @Override
        public Object intercept(MethodInvocationContext<Object, Object> context) {
            Logger log = LoggerFactory.getLogger(context.getDeclaringType());
            log.trace("Executing : {}", context.getExecutableMethod());
            try {
                Object result = context.proceed();
                log.trace("Method {} resulted in : {}", context.getExecutableMethod(), result);
                return result;
            } catch(RuntimeException e) {
                log.trace("Method {} caused error : {}", 
                			context.getExecutableMethod(), 
            				e.getMessage());
                throw e;
            }
        }
    }
    </copy>

A `MethodInterceptor` like the one above intercepts a method invocation and allows you to wrap the invocation in additional functionality before proceeding (by calling `proceed()`). In the above case the method invocation is logged and the result logged or if an error occurs the error message is logged before the error is rethrown.

## Add an Around Advice Annotation

To apply this interceptor to a class you need a trigger annotation. Create a new annotation class in a file called `src/main/java/example/micronaut/Logged.java` with the following contents:

    <copy>
    package example.micronaut;

    import io.micronaut.aop.Around;
    import io.micronaut.context.annotation.Type;

    import java.lang.annotation.Retention;
    import java.lang.annotation.RetentionPolicy;

    @Retention(RetentionPolicy.RUNTIME)
    @Around
    @Type(LoggingInterceptor.class)
    public @interface Logged {
    }
    </copy>

The annotation itself is annotated with [@Around](https://docs.micronaut.io/latest/api/io/micronaut/aop/Around.html) which indicates this interceptor will be applied around a method invocation and the `@Type` annotation is used to indicate the implementation interceptor.

## Apply the Annotation to a Bean

Now apply the `@Logged` annotation to the `getInitialOwners()` method you defined in the previous lab:

    <copy>
    @Logged
    List<Owner> getInitialOwners() {
        return ownerConfiguration.stream()
                 .map(OwnerConfiguration::create)
                .collect(Collectors.toList());
    }
    </copy>

Next modify your logging configuration by modifying the `src/main/resources/logback.xml` file to include the following definition:

    <copy>
    <configuration>
    	<!-- remaining contents omitted for brevity -->

        <logger name="example.micronaut" level="trace" />
    </configuration>
    </copy>

Finally run the `OwnerServiceTest` from the previous lab and you will see the following output when the `testOwners` test is executed:

```
TRACE example.micronaut.OwnerService - Executing : List getInitialOwners()
TRACE example.micronaut.OwnerService - Method List getInitialOwners() resulted in : [example.micronaut.Owner@12294df9, example.micronaut.Owner@60f0698f]
```

You have successfully written your first AOP Around advice! 

You may now *proceed to the next lab*.

### Acknowledgements
- **Owners** - Graeme Rocher, Architect, Oracle Labs - Databases and Optimization
- **Contributors** - Palo Gressa, Todd Sharp, Eric Sedlar