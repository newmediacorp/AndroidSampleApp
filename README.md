# Android Sample Application

This is boilerplate code and Android Studio project for bootstraping Android phone apps. 

It includes the following libraries:

- [Retrofit 1.9.0](http://square.github.io/retrofit/)
- [OkHttp 2.0.0](http://square.github.io/okhttp/)
- [Picasso 2.5.2](http://square.github.io/picasso/)
- [Otto 1.3.8](http://square.github.io/otto/)

##Retrofit

Retrofit turns your REST API into a Java interface.

```
public interface GitHubService {
  @GET("/users/{user}/repos")
  List<Repo> listRepos(@Path("user") String user);
}
```

The RestAdapter class generates an implementation of the GitHubService interface.

```
RestAdapter restAdapter = new RestAdapter.Builder()
    .setEndpoint("https://api.github.com")
    .build();

GitHubService service = restAdapter.create(GitHubService.class);
```

Each call on the generated GitHubService makes an HTTP request to the remote webserver.

```
List<Repo> repos = service.listRepos("octocat");
```

##OkHttp

This program downloads a URL and print its contents as a string. Full source.

```
OkHttpClient client = new OkHttpClient();

String run(String url) throws IOException {
  Request request = new Request.Builder()
      .url(url)
      .build();

  Response response = client.newCall(request).execute();
  return response.body().string();
}
```

####Post to a Server

This program posts data to a service. Full source.
```
public static final MediaType JSON
    = MediaType.parse("application/json; charset=utf-8");

OkHttpClient client = new OkHttpClient();

String post(String url, String json) throws IOException {
  RequestBody body = RequestBody.create(JSON, json);
  Request request = new Request.Builder()
      .url(url)
      .post(body)
      .build();
  Response response = client.newCall(request).execute();
  return response.body().string();
}
```

##Picasso

Images add much-needed context and visual flair to Android applications. Picasso allows for hassle-free image loading in your application—often in one line of code!

```
Picasso.with(context).load("http://i.imgur.com/DvpvklR.png").into(imageView);
```

##Otto

####Introduction

Otto is an event bus designed to decouple different parts of your application while still allowing them to communicate efficiently.

Forked from Guava, Otto adds unique functionality to an already refined event bus as well as specializing it to the Android platform.

####Usage

Otto is designed with Android-specific use cases in mind and is intended for use as a singleton (though that is not required). Create an instance of an event bus with

```
Bus bus = new Bus();
```

Because a bus is only effective if it is shared, we recommend obtaining the instance through injection or another appropriate mechanism.

####Publishing

Event publishing is the most important part of the bus as it allows you to tell subscribers that an action has occurred. An instance of any class may be published on the bus and it will only be dispatched to subscribers for that type.

To publish a new event, call the post method:

````
bus.post(new AnswerAvailableEvent(42));
````

Posting to the bus is a synchronous action so when program execution continues it is guaranteed that all subscribers have been called.

####Subscribing

Subscription is the complement to event publishing—it lets you receive notification that an event has occurred. To subscribe to an event, annotate a method with @Subscribe. The method should take only a single parameter, the type of which will be the event you wish to subscribe to.

To listen for the event published in the previous section we would need the following:

```
@Subscribe public void answerAvailable(AnswerAvailableEvent event) {
    // TODO: React to the event somehow!
}
```

The name of the method can be anything you like. The annotation, single argument, and public accessor are all that is required.

In order to receive events, a class instance needs to register with the bus. If this refers to an instance of the class in which the previous method was present, we can register using the following:

```
bus.register(this);
```

Registering will only find methods on the immediate class type. Unlike the Guava event bus, Otto will not traverse the class hierarchy and add methods from base classes or interfaces that are annotated. This is an explicit design decision to improve performance of the library as well as keep your code simple and unambiguous.

Remember to also call the unregister method when appropriate. See the sample application, available on GitHub, for a complete example.

####Producing

When subscribing to events it is often desired to also fetch the current known value for specific events (e.g., current location, active user, etc.). To address this common paradigm, Otto adds the concept of 'Producers' which provide an immediate callback to any subscribers upon their registration.

To create a producer, annotate a method with @Produce. The method should take no parameters and its return type will be used as the type of event for which it can produce initial values. If we are keeping track of the last answer event somewhere from above we can register to produce its initial value:

```
@Produce public AnswerAvailableEvent produceAnswer() {
    // Assuming 'lastAnswer' exists.
    return new AnswerAvailableEvent(this.lastAnswer);
}
```

Producers, like subscribers, must also be registered:

```
bus.register(this);
```

When registering, the producer method will be called once for each subscriber previously registered for the same type. The producer method will also be called once for each new method that subscribes to an event of the same type.

You may only have one producer per event type registered at a time on a bus.

####Thread Enforcement

Since at times it may be ambiguous on which thread you are receiving callbacks, Otto provides an enforcement mechanism to ensure you are always called on the thread you desire. By default, all interaction with an instance is confined to the main thread.

```
// Both of these are functionally equivalent.
Bus bus1 = new Bus();
Bus bus2 = new Bus(ThreadEnforcer.MAIN);
```

If you are not concerned on which thread interaction is occurring, instantiate a bus instance with ThreadEnforcer.ANY. You can also provide your own implementation of the ThreadEnforcer interface if you need additional functionality or validation.