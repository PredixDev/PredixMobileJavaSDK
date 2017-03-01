# JavaSDK

The Predix Mobile SDK is a comprehensive suite of tools, frameworks and source examples that will enable and educate you on building mobile applications for the Industrial Internet of Things (IIoT). The Java SDK allows you to build desktop client, IIoT screen client or web based applications that are backed by a database that can replicate data to a host server.

For more information about Predix Mobile and available APIs please visit the main [PredixMobileSDK](https://github.com/PredixDev/PredixMobileSDK) repo and [wiki](https://github.com/PredixDev/PredixMobileSDK/wiki)

## Downloading the SDK.

The SDK can be downloaded from the main [PredixMobileSDK repo](https://github.com/PredixDev/PredixMobileSDK) by visiting the [releases tab](https://github.com/PredixDev/PredixMobileSDK/releases).

## Configure Maven Proxy and Enable Annotation Processing

If you are running the Java SDK from Intellij or Android Studio:

1) Configure your maven proxy be adding these command line options to your Maven Importing VM Options (VM Options for importer):
`-Dhttp.proxyHost=proxy-src.research.ge.com -Dhttp.proxyPort=8080  -Dhttps.proxyHost=proxy-src.research.ge.com -Dhttps.proxyPort=8080`

2) Enable annotation processing.

  In the Ribbon menu (If using an IDE):
  
  a. Select IntelliJ IDEA -> Preferences -> Build`.
  
  b. Select `Execution`.
  
  c. Select `Deployment -> Compiler -> Annotation Processors -> Enable annotation processing`.
  

## Create a Custom Service

The Predix Mobile SDK includes the Mobile Client Core Services Framework, a set REST APIs that provide functionality to hybrid or native Mobile apps. In addition to the services in the Mobile Client Core Services Framework, you can create a custom service. The Client SDK consuming application (Predix Mobile App Container) can interact with these services following this general URL structure: `http://pmapi//<parameters>`.

Custom services allow you to extend the REST interface that Predix Mobile provides to a web view. For example, a custom service could provide native integration with a custom tool, such as a Bluetooth sensor.  This would allow a hybrid application to access that sensor from a web application using a standard REST or ajax request.  

Here is an example of a simple custom service that you could create with the Predix Mobile Java SDK. With the following code, your web app could send a REST or ajax request to `https://pmapi/sensor/data` to retrieve data from a Bluetooth sensor:

```
@RequestMapping("sensor")
public class CustomService {
    @RequestMapping("data")
    @RequestCompleteOnReturn
    public void readSensorData() {
        //code to read sensor data from Bluetooth
    }
}
```
## Register a Custom Service
You must register your custom service with the SDK's service router: 

```
mobileManager.getServiceRouter().registerService(yourCustomServiceClass.class);
```
 
## Service Annotations

Predix Mobile allows you to define and register custom services that are used to extend the functionality of Predix Mobile.  The service-based architecture allows your hybrid apps to request resources as they do for any other REST-based request.  The following Java annotations enable you to tell the SDK how your service should work and when it should be called:

- `@RequestMapping`
- `@RequestParam`
- `@PathVariable`
- `@RequestCompleteOnReturn`

By default we handle certain errors conditions. For example, if a service and endpoint can't be identified, a 404 status code is returned in the response.  For another example, if you configure your service to only support GET requests but another command is sent, a 405 status code is returned in the response.  

### About the `@RequestMapping` Annotation 

Use the `@RequestMapping`annotation to map classes and methods to your custom service.

### - `@RequestMapping` Class-level

When the `@RequestMapping` annotation appears above a class, that class is mapped to the service class that you registered.

For example, the following code routes `https://pmapi/myServiceClass/` to the `CustomService` class.
```
@RequestMapping("myServiceClass")
public class CustomService {}
```

### - `@RequestMapping` Method-level

When the `@RequestMapping` annotation appears above a method, that method is mapped to service method, 

For example, the following code routes `https://pmapi/myServiceClass/endpoint` to the service's `endpointMethod`:
```
@RequestMapping("myServiceClass")
public class CustomService {
    @RequestMapping("endpoint")
    public void endpointMethod() {
        //...
    }
}
```


### - `@RequestMapping` Method-level Mapping Options

Use the `@RequestMapping` annotation method parameter to map a class to a specific HTTP method.  For example, the following code routes `GET` requests to `doGET()`, and `POST` requests to `doPost()`.  (If the method parameter is not set, by default it maps that request to `GET`.) **<<WHAT MAPS THE REQUEST TO GET. FINAL SENTENCE UNCLEAR.>>**

```
@RequestMapping("myServiceClass")
public class CustomService {
    @RequestMapping("endpoint")
    public void doGET() {
        //...
    }
    
    @RequestMapping(value = "endpoint", method = RequestMethod.POST)
    public void doPOST() {
        //...
    }
}
```

You can group request methods by encasing them in brackets, for example:

```
@RequestMapping("myServiceClass")
public class CustomService {
    @RequestMapping(value = "endpoint", method = {RequestMethod.GET, RequestMethod.POST})
    public void doPOST() {
        //...
    }
}
```

### - `@RequestMapping` Method-level Annotation Options

- `value`:  The path for the endpoint.  The `value` option has three sub-options: `static path name`, `path variable`, and `path wild card`.

- `method`:  The HTTP method your endpoint is expecting: `RequestMethod.GET`, `RequestMethod.POST`, `RequestMethod.PUT`, `RequestMethod.DELETE`.  

   Methods can be chained togeather so one method can handle multiple HTTP Methods.  If a given request does not match the endpoint's supported methods the SDK will return the request with a 405 status code (Method Not Supported).


### - `@RequestMapping` Method-level Annotation Value Options

For your endpoint path/value you can use some special characters to accept things like path variable and wild card statements.  Here are the options you can use to specify your service endpoint:

- `Full/Exact path`:  This option tells the SDK to expect an exact path such as `/customService/start`.  The router will only call this service endpoint if the REST request URI exactly matches the `@RequestMapping` (`https://pmapi/customService/start` in this example).

- `PathVariable`: This option enables you to take a section of the request path as a variable in your endpoint method.  
  Consider this REST request example:  `https://pmapi/user/12345`. In this case, you would want your service to be able to take `12345` as a variable in your `endpoint` method.  The SDK enables you to do this by giving a special designation to the `@RequestMapping` annotation.  

Specify the path of a `@RequestMapping` with brackets `{}`, such as `https://pmapi/user/{id}`, to tell the SDK there will be a path variable in the request path.  See the `@PathVariable` explanation below for more information.

- `wild card`:  Wild cards allow you to take more URI paths than you specified in your request mapping.  For example, if you specify a request mapping as `@RequestMapping("1/2/*")` you could send a request like this `https://pmapi/customService/1/2/3` and match to the method that takes `"1/2/*"`.  

  **WARNING:** When a wildcard is specified at the base level of a path for a method (for example `@RequestMapping("*")`), the wild card method is matched anytime the SDK fails to match the other mapped methods in the service.  This prevents the SDK from rejecting a request on a matching-basis since the wild card service says it can take anything (in this example, everything that is a `GET` request).  Due to this behavior, it is important to consider all failure scenarios when using a wild card in the path, and respond to the caller accordingly if the service endpoint receives input it was not expecting.
  
- `wild card PathVariable`:  The wild card PathVariable is a combination of the `wild card` option and the `PathVariable option`.  
  For example, `@RequestMapping("/1/{*}")` returns all other path items as a string.  In this example, `https://pmapi/customService/1/2/3` is passed to a `@PathVariable String restOfPath` as a string with `"2/3"` as the value of `restOfPath`.

### - ```@RequestMapping``` Order of Operations

The SDK uses the following order of operations to determine when a request URI should match a `@RequestMapping`:  

1. `Full/Exact path`
2. `PathVariable`
3. `wild card PathVariable`
4. `wild card`

## About the `@RequestParam` Annotation

Use the `@RequestParam` annotation to tell the SDK that you expect or optionally need a parameter from the request.  The request parameter can come from one of three locations: `body`, `query`, or `headers`).  By default, the SDK examines all three locations for a request parameter. However, you can restrict the search locations.  

```@RequestParam``` can be be added to the methods parameter list and you can have as many as you need.  By default a ```@RequestParam``` is assumed to be required but they can be made optional.

Here are a couple of examples:

```
@RequestMapping("myServiceClass")
public class CustomService {
    @RequestMapping(value = "endpoint")
    public void doPOST(@RequestParam("item") String item, @RequestParam(value = "aHeaderKey", location = ParamLocation.HEADER, required = false) {
        //...
    }
}
```

### `@RequestParam` Options

- `value`:  The key of the parameter
- `location`: Where to look for the key of the parameter (`ParamLocation.QUERY`, `ParamLocation.HEADER` or `ParamLocation.BODY`).  
  If no `location` is given, by default the SDK looks in all three locations.
- `required`:  Tells the SDK to give a 400 (bad request) response to the caller if the parameter is not found in the given locations.

## About the `@PathVariable` Annotation 
**Check this section**

Use the `@PathVariable` annotation to take a section of the request path as a variable in your endpoint method.  

Specify the path of a `@RequestMapping` with brackets `{}`, such as `https://pmapi/user/{id}`, to tell the SDK there will be a path variable in the request path. This indicates that you want your service to take `id` as a variable in your endpoint method.      

Here is an example of a service that uses a `@PathVariable` annotation:

```
@RequestMapping("myServiceClass")
public class CustomService {
    @RequestMapping("user/{id}")
    public void user(@PathVariable String id) {
        //id will contain whatever the request path has at the end of the URI (https://pmapi/user/12345 yields 12345 as the string for id)
        //...
    }
}
```

## About the `@RequestCompleteOnReturn` Annotation 

Use the `@RequestCompleteOnReturn` annotation to tell the ServiceRouter that the request should be marked completed and returned to the caller when a method/service endpoint has finished execution. This can be useful if a service does not return data in the body of the response.  Here is an example:

```
@RequestMapping("myServiceClass")
@RequestCompleteOnReturn
public class CustomService {
    @RequestMapping(value = "endpoint", method = RequestMethod.DELETE)
    public void doDELETE() {
        //...
        if (!database.delete(xyz)) {
            throw new InternalServerException("could not delete")'
        }
        //if no exception is thrown then a 200 will be given to the caller when the method returns.
    }
}
```


