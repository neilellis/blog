title: Introducing Dollar
tags:
    - dollar
    - java
---

I'm currently working on a new library called [Dollar](http://github.com/cazcade/dollar) and I'd like to share some thoughts about it. It is pre-alpha software which means it's really in a prototyping state and the interfaces are not fixed yet.

If you like the ease of JavaScript, Ruby, Groovy etc. but also enjoy being able to work within the Java language then Dollar is for you. You can write typesafe code and then drop into typeless Dollar code whenever you need to. Dollar is both an alternative paradigm and a complementary resource.


So let's create some JSON in Java

``` Java
var profile = $(
        $("name", "Neil"),
        $("age", new Date().getYear() + 1900 - 1970),
        $("gender", "male"),
        $("projects", $jsonArray("snapito", "dollar")),
        $("location",
                $("city", "brighton"),
                $("postcode", "bn1 6jj"),
                $("number", 343)
        ));
```
As you can see, creating nested JSON data is incredibly easy.

<div class="alert alert-info" role="alert">
You're probably wondering about `var profile` in the code above. `var` is the interface name of the dynamic typing systems. It was chosen to help the code read more like a dynamic language.
</div>


And then we can query that JSON using Dollar, in much the same way you would in jQuery.

``` Java
String name = profile.$("name").$$();
```

Or we can do queries using JavaScript (Nashorn) expressions ($ means the current value).

``` Java
profile.$("$['age']/11").$int()
profile.$("$.gender").$()
```

We can also create our objects from JSON Strings ...
``` Java
$("{\"name\":\"Dave\"}").$("name").$$()
```

Lists ...

``` Java
list = $list("Neil", "Dimple", "Charlie");
assertEquals(list, $list("Neil").add("Dimple").add("Charlie"));
```

Or maps ...

``` Java
Map map = new HashMap();
map.put("foo", "bar");
Map submap = new HashMap();
submap.put("thing", 1);
map.put("sub", submap);
assertEquals(1, $(map).$("sub").$map().get("thing"));
```

Dollar has built in support for being a webserver (using [Spark](http://www.sparkjava.com/)):

``` Java
//Serve up the request headers as a JSON object under the /headers URL
$GET("/headers", (context) -> context.headers());
```

And also supports queues:


``` Java
profile.push("test.profile");
var deser = profile.pop("test.profile", 10 * 1000);
```

... persistence ...

``` Java
assertTrue(profile.save("test.profile.set").equals(profile));
var deser = profile.load("test.profile.set");
Assert.assertEquals(deser.$$(), profile.$$());
```

... and pub/sub

``` Java
final int[] received = {0};
Sub sub = $sub((message) -> {
    received[0]++;
}, "test.pub");
sub.await();
profile.pub("test.pub");
Thread.sleep(100);
sub.cancel();
assertEquals(1, received[0]);
```

JSON Friendly
=============

Dollar can work with potentially any data format but the core features of dollar revolve around a JSON centric world view. This means working with JSON as you can see in the above example is very easy indeed.


<!-- more -->


For Real Applications
=====================

Dollar is designed for production, it is designed for code you are going to have to fix, debug and monitor. Every library and language has it's sweet spot. Dollar's sweetspot is working with schema-less data in a production environment. It is not designed for high performance systems (there is a 99.9% chance your project isn't a high performance system) but there is no reason to expect it to be slow either. Where possible the code has been written with JVM optimization in mind.

Characteristics
===============

> "The secret of success is to be like a duck – smooth and unruffled on top, but paddling furiously underneath.” - Anon

* Simple - Dollar does do not expose unnecessary complexity to the programmer, we keep it hidden.
* Typeless - if you *need* strongly typed code stop reading now. If you're writing Internet-centric and modest-sized software this is unlikely to be the case.
* Synchronous - asynchronous flows are hard to follow and even harder to debug in production. We do not expose asynchronous behaviour, where possible, to the programmer.
* Metered - key execution's are metered using Coda Hale's metrics library, this makes production monitoring and debugging easier.
* Nullsafe - Special null type reduces null pointer exceptions, which can be replaced by an isNull() check.
* Threadsafe - No shared state, always copy on write. No shared state means avoidance of synchronization primitives, reduces memory leaks and generally leaves you feeling happier. It comes at a cost (object creation) but that cost is an acceptable cost as far as Dollar is concerned.

