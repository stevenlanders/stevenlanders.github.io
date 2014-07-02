---
layout: post
title:  "Spring AOP - invoke when method has annotation"
date:   2014-07-02 23:09:25
categories: jekyll update
---

This is a quick example for how to run a method when any invoked method has a particular annotation.  In this example, we will log the time it takes to execute any method adorned with a `@LogDuration` annotation.  (Why log duration?  You may wish to do something else.  This is fine - just do that instead.)

###1. Tell spring to use AOP and annotations.  

In this case, we're using `<aop:aspectj-autoproxy>`.

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-3.1.xsd
           http://www.springframework.org/schema/aop 
       http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc-3.0.xsd">

  <!-- Scans for application @Components to deploy -->
    <context:annotation-config/>
  <context:component-scan base-package="com.stevenlanders.*" />

  <!-- enable AOP -->
  <aop:aspectj-autoproxy/>

</beans>
{% endhighlight %}

###2. Create your desired annotation

{% highlight java %}
@Retention(RetentionPolicy.RUNTIME)
public @interface LogDuration {
   String value();
}
{% endhighlight %}

###3. Write your AOP logic

In this case, we should choose the `@Around` annotation.  This is because we need to do something before AND after the method invocation.

{% highlight java %}
@Aspect
public class AopExample{

    //for any method with @LogDuration, no matter what the return type, name, or arguments are, call this method 
    @Around("execution(@com.stevenlanders.annotation.LogDuration * *(..)) && @annotation(logDurationAnnotation)")
    public Object logDuration(ProceedingJoinPoint joinPoint, LogDuration logDurationAnnotation) throws Throwable {
       
        //capture the start time 
        long startTime = System.currentTimeMillis();
        
        //execute the method and get the result
        Object result = joinPoint.proceed();
        
        //capture the end time
        long endTime = System.currentTimeMillis();
        
        //calculate the duration and print results
        long duration = endTime - startTime;
        System.out.println(logDurationAnnotation.value()+": "+duration+"ms"); //you should use a logger  
        
        //return the result to the caller
        return result; 
    }

}
{% endhighlight %}

###4. Toss your annotation on a method.

{% highlight java %}
@Controller
public class ExampleController{

     @RequestMapping("/api/example/hello")
     @LogDuration("Hello World API") 
     public @ResponseBody String getHelloWorld(){
          try {
            Thread.sleep(3000);
          } catch (InterruptedException e) {
            throw new RuntimeException("Sleep Interrupted", e);
          } 
          return "Hello World";
     } 

}

{% endhighlight %}



Check out the [Jekyll docs][jekyll] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll's GitHub repo][jekyll-gh].

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com
