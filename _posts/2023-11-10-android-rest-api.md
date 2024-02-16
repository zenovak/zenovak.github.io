---
title: Android - RestAPI
date: 2023-11-10
categories: [Android, Libraries, Cheatsheets]
tags: [android, restapi, java]     # TAG names should always be lowercase
---


In order to connect to a web service and get resful api responses we need to use the **Retrofit** library.

Making API requests using retrofit requires a little more verbose configurations compared to a typical pythonic way. The retrofit library only handles the complexity of making requests, lower level networkings etc. Not the data convertion

We need to convert a Java object into JSON and vice versal when communicating with a HTTP server, to do that we use the ***GSON library.***


### Requirements

- Retrofit
- [gson](https://github.com/google/gson)


### Dependencies
You need to add the dependencies into your gradle build files and configurations.
```gradle
dependencies {
    implementation 'com.google.code.gson:gson:2.8.9'
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.9.0'
}
```

### Permissions
You will of course need to handle the permission requests inside your application's `onCreate()` method.

```
<uses-permission android:name="android.permisson.INTERNET">
```

<br><br>

---
## Using Retrofit

There are 3 components you need to facilitate an effective commuinication:
- Retrofit Instance
- Interface
- Model class

The retrofit instance works as a singleton. Meaning we only use *1 instance of retrofit per Endpoint* throughout the entire lifecycle of our application.

To send a GET requests to a HTTP server, we need to define 2 important components:
- The request `interface`
- The response `model` class

The request interface would act as an async template of our request payload, while the model class would "model" the expected response data.

<br>

### Model Class
You have to know before hand, the exact type and format of the JSON response you expected to receive from the API. Here's an example scenario:

Expected JSON payload response:
```json
{
    "user_name": "John",
    "user_id": 1,
    "user_friends": [2, 3, 4]
}
```

So your model class would look like:
```java
public class Example {
    private String user_name;
    
    private Integer user_id;
    
    private Integer[] user_friends;
    ...
}
```

<br>

However, we all know how some JSON formats are a lot more complicated than this, nested lists and objects within lists. Here's where online tools such as jsonpojo comes in handy. 

Use them [here](https://www.jsonschema2pojo.org/). Set output to java class and your expected data response in ***GSON*** annotation.

<br><br>

---

## GSON
GSON is a google library that will perform the Java objects to JSON representation conversion. As much as we could say all we needed to do was parse a string. Gson handles that mess for us and provides a handy `toJson()` and `fromJson()` methods.

Using JSONpojo, we get the following Gson representation of our class:
```java
package com.example;  
  
import java.util.List;  
import javax.annotation.Generated;  
import com.google.gson.annotations.Expose;  
import com.google.gson.annotations.SerializedName;  
  
@Generated("jsonschema2pojo")  
public class Example {  
  
@SerializedName("user_name")  
@Expose  
private String userName;  
@SerializedName("user_id")  
@Expose  
private Integer userId;  
@SerializedName("user_friends")  
@Expose  
private List<Integer> userFriends;  
  
public String getUserName() {  
return userName;  
}  
  
public void setUserName(String userName) {  
this.userName = userName;  
}  
  
public Integer getUserId() {  
return userId;  
}  
  
public void setUserId(Integer userId) {  
this.userId = userId;  
}  
  
public List<Integer> getUserFriends() {  
return userFriends;  
}  
  
public void setUserFriends(List<Integer> userFriends) {  
this.userFriends = userFriends;  
}  
  
}
```

<br>

Point your attention to these annotations:
```java
@SerializedName("user_id")
@Expose
private Integer userId;
```

`@SerializedName()` annotates the parser to parse this field's values into json with key set to "user_id".

`@Expose` This annotation is optional. We can see them in the wild as either `@Exposed(serialize = false)` or `@Exposed`. When given the `serialize = false` parameter, this field will be ignored by the parser. 

<br><br>

---
## The interface
The Retrofit interface is what we will use to perform the GET, POST API requests. We often name it with a *service* suffix. 

Each method within this service interface can be used to represent an API endpoint path from a baseURL.

<br>

### GET Requests
Let's say this is our url endpoint: `http://myendpoint/user`. We will only use the specific API endpoint for the interface (`user`), the base URL `http://myendpoint/` will be declared in the `Retrofit` instance.

So first we must create an interface to represent the request params
```java
import retrofit2.http.GET;


public interface GetUserDataService {

    @GET("user")
    Call<Example> getUser();
}
```

`Call<T>` class from retrofit represents the HTTP request object. The `T` generic here represents the expected response body. 

`@GET` annotation indicates that this request is a GET request, and the `"user"` part is the endpoint.

The requests payload is modelled as part of the interface's params. Here's another example:
```java
import retrofit2.http.GET;


public interface GetDataService {

    @GET("user")
    Call<ResponseBody> getEnedisData( 
        @Query("year") String year, 
        @Query("month") String month, 
        @Query("line") String line, 
        @Query("plot_type") String plotType, 
        @Query("unit") String unit 
    );
}
```

<br>

### POST Requests
On a `POST` request, The JSON payload will be sent as part of the `@Body` params instead. A template model class must be crafted. 

```java
import retorfit2.http.POST;

public interface ApiService {

    @POST("/api/enedis")
    Call<ResponseBody> postEnedisData(@Body EnedisRequest request);
}
```

The model request body class will be parsed and converted also via `GSON` into a JSON payload. So `GSON` synthaxes do apply :
```java
import com.google.gson.annotations.SerializedName;  


public class EnedisRequest { 
    @SerializedField("year")
    private String year; 
    
    @SerializedField("month")
    private String month; 
    
    @SerializedField("line")
    private String line; 
    
    @SerializedField("plot_type")
    private String plotType; 
    
    @SerializedField("unit")
    private String unit; 
    
    // getters and setters
    ...
}
```

As we can see, if an API endpoint expects a JSON payload, then we must prepare a model class. It gets more complicated when we have nested JSON parameters such as a list of elements or another object. These expectations must be met with a request/response model class equivalent.
```json
{
    user_id: "123",
    friends: ["John", "Link"]
}
```

<br>

> **Notice naming convention**\
> In the interface the methods should be named as `postData` or `getData` according to the API requests of type `POST` or `GET`
{: .prompt-info}

<br>

### Requests with variable paths
Let's assume the following API endpoint. This endpoint has a variable path, `id_number`
```python
# sending an SDP offer to a number 
@app.post("offers/{id_number}") 
def post_offers(id_number, sdp_document: SDPDocument):
    _offers[id_number] = sdp_document 
    return {"status": "complete"}
```

We can just as easily create a request towards this endpoint in our retrofit interface:
```java
public interface SendOfferService {
    @POST("offers/{id_number}")
    Call<Response> sendSDPOffer(@Path("id_number") String idNumber, @Body SDPDocument sdpDocument);
}
```

`{id_number}` is a placeholder for the actual ID number variable path

`@Path("id_number")` annotation binds the `idNumber` method params to the `{id_number}` placeholder in the URL.

<br><br>

---
## Retrofit Instance
Fundamentally, to prepare the Retrofit intance for sending an API request, requires 2 build steps: 
- the retrofit intance
- the service instance

Create a `Retrofit` instance to represent the API base endpoint. 
```java
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://myendpoint/")
    .addConverterFactory(GsonConverterFactory.create())
    .build();
```

Create an Instance of the API service:
```java
ApiService apiService = retrofit.create(ApiService.class);
```

<br>

Since only **1 instance** of` RetrofitService` is needed per base endpoint, a `RetrofitService` singleton class can be made as follows:
```java
public class RetrofitInstance {
    private static Retrofit retrofit = null;
    private static String BASE_URL = "https://myendpoint/";
    
    // constructors etc..

    public static APIService createService() {
        if (retrofit == null) {
            retrofit = new Retrofit
                .Builder()
                .baseUrl(BASE_URL)
                .addConverterFactory(GsonConverterFactory.create())
                .build();
        }
        return retrofit.create(ApiService.class)
    }
}
```
However, there are intances where a lifecycle bound singleton isnt desired. Such as allowing changes to the baseURL in the app settings. 
***Rebuilidng a new instance is preferable when the baseURL is adjustable.***

<br>

### Making a request
We prepare the request body. This **does not start an Async request or action**, its just a request body.
```java
APIService service = RetrofitInstance.createService();
Call<ResponseBody> call = service.getEnedisData(
    "2023", 
    "November", 
    "someLine", 
    "bar", 
    "kWh"
);
```
Where, `ResponseBody` represents the model data of the API response.


The actual request made is `async`, with an `enqueue` call. 
```java
// Async non-blocking
call.enqueue(new Callback<ResponseBody>() {
    @Override
    public void onResponse(Call<ResponseBody> call, Response<ResponseBody> response) {
        
        if (response.body() != null) {
            ResponseBody responseBody = response.body();
            // Do things with responseBody model class
        }
    }
    
    @Override
    public void onFailure(Call<ResponseBody> call, Throwable t) {
        // Handle the failure
    }
});
```

`null` checks are neccessay or the application will crash, as API requests dont trigger an `onFailure` when something with the server goes wrong.

Alternatively, a synchronous operation is as follows:
```java
try {  
    Response<CaptionModel> response = call.execute();     // Blocking operation
    if (response.isSuccessful() && response.body() != null)  
        return response.body().caption;  
    else        
        Log.i("ImageAnalysis", "API Error, please check server");  
  
} catch (IOException e) {  
    e.printStackTrace();  
}  
return "API Error";
```

<br><br>

---
## Network Errors:


### Android 8: Cleartext HTTP traffic not permitted
> [Stack overflow](https://stackoverflow.com/questions/45940861/android-8-cleartext-http-traffic-not-permitted)

Starting with Android 9 (API level 28), cleartext support is disabled by default. This basically means you cannot send Restful API request via HTTP, it must be HTTPS. 

You can change your endpoint from: `http://mysite.com/api/` to `https://mysite.com/api/`. However if it does not work, you must configure network security to allow HTTP

Declare a network security configuration as follows:

Inside `res/xml/network_security_config.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <!--Set application-wide security config using base-config tag.-->
    <base-config cleartextTrafficPermitted="false"/>
</network-security-config>  
```

Then declare this file inside manifest:
```xml
.
.
.
<application
    ...
    android:networkSecurityConfig="@xml/network_security_config" >
    .
    .
    .
</application>
```

<br><br>

---
# References

https://howtoprogram.xyz/2017/02/17/how-to-post-with-retrofit-2/

https://square.github.io/retrofit/



