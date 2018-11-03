# iRequest
Thinnest HTTP Client ***based on HttpURLConnection*** provides you - wraps all functionality in soft manner with fluent-like interface.

This client abstracts you from all boilerplate and extra dependency(JSON, XML etc.) that you write within native API as hard as possible.    

 
 ```xml
 <dependency>
     <groupId>com.github.timabilov</groupId>
     <artifactId>irequest</artifactId>
     <version>1.0.3</version>
 </dependency>
 ```   


**DO NOT** supports get requests with body  because of specification and etc.

http://stackoverflow.com/questions/18664413/can-i-do-an-http-get-and-include-a-body-using-httpurlconnection






                    

Stream response to anywhere directly or store after

```java

Request.get("http://httpbin.org/ip")
        .proxy("27.48.5.68", 8080)
        .pipe(System.out);
        
Request.get("https://www.google.az/favicon.ico?key=initial")
        .arg("key2", "additional") // url args
        .snapshot() // prints request before send
        .pipe("favicon.ico");

Request.get("https://www.google.az/")
        .header("User-Agent", "iRequest Agent") // it's default user-agent
        .snapshot() // prints request before send
        .send() // Response object
        .printHeaders()
        .store("google.html");
```

Use `fetch()` to get result immediately. Also may use ` fetchJson()` instead. 

```java
JsonNode json  = Request.get("http://httpbin.org/basic-auth/username/password123")
        .basicAuth("username", "password123")
        .timeout(10) //read and connect timeout
        .fetchJson(); // result as json node

Request.json("http://httpbin.org:80/put", Method.PUT)
        .param("key", "value")
        .pipe("put.json");
```

Handle fired requests asynchronously(not at I/O level) : 

```java
Request.get("http://httpbin.org/hidden-basic-auth/user/passwd")
        .async(new ResponseHandler() {

            public void success(Response r) {
                System.out.println("Async fetch!");
                System.out.println(r.getBody());
            }

            public void error(BrokenRequestException failedRequest) {
      
                try {
                    failedRequest.repair()
                            .header("ThisHeaderChangesEverything", "Really")
                            .basicAuth("user", "passwd")
                            .pipe("repaired.json");

                } catch (Exception e){
                    e.printStackTrace();
                    System.out.println("Damn!");
                }
            }
        });
```
      

`jsonify()` will adapt your form request with params to json request  
 
```java
 String result = Request.post("http://www.posttestserver.com/post.php")
             .param("name", "John")
             .jsonify() // { "name": "John" }
             .fetch()
```

Post request built with both form and query params:
 
``` java
    
Request.post("http://www.posttestserver.com/post.php")
        .header("Header", "Header-Value")
        .param("formParam", "formValue")
        .arg("urlParam", "UrlValue")
        .snapshot() // prints raw request
        .pipe("post.txt");
```

You can convert post request with converted params from plain form to multipart and json respectively   

```java
result = Request.post("http://www.posttestserver.com/post.php")
        //.proxy("112.214.73.253", 80)
        .param("phone", "+994XXYYYYYYY")
        .param("id", "123456")
        .param("file", new File("C:/Finish.log")) // Upload file. Implicitly casts to multipart(!).
        .jsonify() // force convert to json request(not multipart anymore) with file translation encoded
        // BASE64 body { ... "file":{"name": "Finish.log", "body": "RmluaXNoIA0K"}}
        .snapshot()
        .fetch();
```

Proxy can be set locally or globally.

```java
String result = Request.post("http://www.posttestserver.com/post.php")
        .proxy("27.48.5.68", 8080, true) // you can set one-time like this - and also do like down below
        .saveProxy()  // save this request proxy settings globally until overwritten
        .param("name", "John")
        .jsonify() // previous params also converted to json
        .param("jsonKey", MapUtils.mapOf("nestedKey", "nestedValue"))
        .fetch();


Request.get("httpbin.org/ip")
        .pipe(System.out); // Prints last saved proxy


Request.forgetProxy(); //reset global proxy

Request.get("httpbin.org/ip")
        .pipe(System.out); // Prints our origin.
```

Fail silently with bad http codes. No http related exception will be thrown. Instead error body will be considered with response : 

```java
Request.get("http://httpbin.org/status/fake") // return 500
        .suppressFail()
        .timeout(10)
        .pipe("failed.html");
```
            
By default handles gzipped and deflated content. You can set your decode provider `withReader(YourInputStream.class)` which used only if none of the available are fit.

```java
Request.get("http://httpbin.org/gzip")    
        .pipe("gzipped.json");
```

Log you responses, store somewhere and etc.

```java
Request.get("http://httpbin.org/response-headers")
        .arg("arg", "argValue")
        .send()
        .dump("log.txt", true) // appends log/dump response metadata
        .store("response.json")
        .printHeaders();
```

Set body directly. Not allowed with get requests.

```java
result = Request.post("http://httpbin.org:80/post")
        .cookie("name", "3")
        .body("Native content") 
        .cookie("lang", "en")
        .snapshot()
        .fetch();
```

By default cookies are validated. You can disable this `Request.skipCookieValidation(true)`

