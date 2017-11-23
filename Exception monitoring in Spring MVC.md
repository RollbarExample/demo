Exception monitoring in Spring MVC

The [Spring Famework](https://spring.io/) is the most popular framework for Java according to [hotframeworks.com](http://hotframeworks.com/languages/java). It provides a model view controller (MVC) architecture and readily available components to develop flexible and loosely coupled web applications. 

If you are new to Rollbar, it helps you monitor errors in real-world applications. It provides you with a live error feed from the application, including complete stack traces and contextual data to debug errors quickly. It lets you easily understand user experience by tracking who is affected by each error. Learn more about our [product features for Java](https://rollbar.com/error-tracking/java/).

While Rollbar’s notifier works with any Java application, we’re going to show you how to set it up with Spring and how to try it out yourself with a working example app. 

## Create a global exception handler

To track all of our exceptions in Spring, we’ll be making use of a global exception handler. This receives uncaught exceptions for your whole application, not just an individual controller. Spring offers two main approaches:

#### 1. ControllerAdvice

When you create a class annotated with [@ControllerAdvice](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc), it will handle exceptions created by all your controllers. Each controller advice defines a method with a @ExceptionHandler annotation which becomes the default handler. You can insert your custom code to print or track errors there.

ControllerAdvice is only available in Spring 3.2 and above. We won’t be covering this approach in detail but you can see our working [example on GitHub](https://github.com/RollbarExample/Rollbar-Java-Example/blob/master/src/main/java/com/in28minutes/controller/GlobalExceptionHandlerController.java). You will need to uncomment the annotation at the top to run it.

#### 2. HandlerExceptionResolver 

Spring Framework also provides a HandlerExceptionResolver interface that you can implement to create a global exception handler. Since it has been around since early releases, it should be compatible with both old and new code.The Spring Framework makes it easy to set up by providing a SimpleMappingExceptionResolver. We’ll show you how to override it to create your own global handler with exception tracking. 

**Warning:** Be careful about mixing both approaches in the same application. Most applications use one approach, and using two may result in unexpected behavior.

The example below shows you how to override the SimpleMappingExceptionResolver. It allows you to create a custom method to build a log message and to return a view to the user with a more friendly error page. If you want to run this example yourself, check out [Rollbar-Example-Java](https://github.com/RollbarExample/Rollbar-Java-Example) on GitHub.

<table>
  <tr>
    <td>public class MyMappingExceptionResolver extends SimpleMappingExceptionResolver {
	
	public MyMappingExceptionResolver() {

	    setWarnLogCategory(MyMappingExceptionResolver.class.getName());
	}

	@Override
	public String buildLogMessage(Exception e, HttpServletRequest req) {
                 
                 System.out.println("Exception : "+e.toString());
	     
                 return "MVC exception: " + e.getLocalizedMessage();
	}
	    
	 @Override
	protected ModelAndView doResolveException(HttpServletRequest req,
	    
                HttpServletResponse resp, Object handler, Exception ex) {
	    ModelAndView mav = super.doResolveException(req, resp, handler, ex);     
	    mav.addObject("url", req.getRequestURL());
    
	    return mav;
            }
}</td>
  </tr>
</table>


In order to make use of this class, you must configure it in your bean configuration file. We also map in a default error page called "error" and pass in the exception attribute, which will give our view access to the exception object for reporting.

<table>
  <tr>
    <td><bean id="simpleMappingExceptionResolver"     class="com.in28minutes.controller.MyMappingExceptionResolver">
   	 <property name="exceptionMappings">
   		 <props>
   			 <prop key="java.lang.Exception">error</prop>
   		 </props>
   	 </property>
   	 <property name="defaultErrorView" value="error" />
   	 <property name="exceptionAttribute" value="ex" />
    </bean></td>
  </tr>
</table>


## Add Rollbar error monitoring

Now that we have a simple exception handler wired up, we’re going to show you how to integrate [Rollbar’s Java Notifier SDK](https://rollbar.com/docs/notifier/rollbar-java/) in your code. 

1. Visit [https://rollbar.com](https://rollbar.com) and sign up for an account if you haven’t done so yet. Next, create your project and select Java  from the list of notifiers. Select the server side access token that is generated for you. You’ll need this to configure Rollbar in the steps below.

2. Add a dependency for the rollbar-web notifier in your chosen package management system. For Maven, add the dependency below in your pom.xml file:

	

<table>
  <tr>
    <td><dependencies>
            <dependency>                <groupId>com.rollbar</groupId>                <artifactId>rollbar-web</artifactId>                <version>1.0.0-beta-3</version>            </dependency></dependencies></td>
  </tr>
</table>


          

For Gradle:

<table>
  <tr>
    <td>compile('com.rollbar:rollbar-web:1.0.0-beta-3')</td>
  </tr>
</table>


3. Now we will add Rollbar tracking inside the MyMappingExceptionResolver that we created earlier. It should go in the buildLogMessage method. Add the access token that we got in step one. We are also passing in the request provider object so we have more context for debugging problems later.

	

<table>
  <tr>
    <td>@Override
public String buildLogMessage(Exception e, HttpServletRequest req) {

    RequestProvider requestProvider = new RequestProvider
        .Builder().userIpHeaderName(req.getRemoteAddr())
        .build();

    rollbar = Rollbar.init(withAccessToken(accessToken)
        .request(requestProvider)
	    .build());
    rollbar.error(e);

    return "MVC exception: " + e.getLocalizedMessage();
}</td>
  </tr>
</table>


You can also use Rollbar to track caught exceptions, warnings, and other items using the same rollbar object. Learn more about the full API in our [documentation](https://rollbar.com/docs/notifier/rollbar-java/). 

## Test with an example app

To test that it’s working, let’s create a page that will generate an error message. In the example below, you can generate an error by clicking the "Throw an error" button. To run the code, just check out [Rollbar-Example-Java](https://github.com/RollbarExample/Rollbar-Java-Example) on GitHub. Just clone the project and replace the access token with your project access token. You can find the access token inside Global Exception Handler class.

![image alt text](image_0.jpg)

This form add a button which will call /spring-mvc/createException.

<table>
  <tr>
    <td><form action="/spring-mvc/createException" method="POST">
    <center>
        <input style="height:50px;width:200px" type="submit"  value="Throw an error" />
    </center>
</form>     	 
</td>
  </tr>
</table>


When you click the "Throw an exception" button, it will trigger the throwException method. In this method, we have added a bug which attempts to call a method on a null object.

<table>
  <tr>
    <td>@RequestMapping(value = "/createException", method = RequestMethod.POST)
public String throwException(ModelMap model) {

    System.out.println("Error : here....");
    String exception = null;
    exception.toCharArray();
   	 
    return "error";
}</td>
  </tr>
</table>


 

Since we wired up our MyMappingExceptionResolver to report the error to Rollbar, we should see the error show up on the Rollbar’s [item page](https://rollbar.com/demo/demo/items/).

![image alt text](image_1.png)

Click on that item to see extra detail on the error. This provides more context to help you find the cause of the error. The Occurrences tab will show the request object that we provided, including the IP address of the client, which browser and operating system they were using, and more. The People tab will show which user was affected and how many times.

![image alt text](image_2.png)

It’s pretty easy to get error tracking across your entire application thanks to Rollbar and Spring’s global exception handlers. It only takes a few minutes to set up, and you will have way more context to track and debug problems faster in the future.

