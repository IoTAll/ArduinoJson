---
layout: doc
tags: doc
title: Encoding JSON
keywords: ArduinoJson,serialize,encode,generate
description: How to generate JSON on Arduino with ArduinoJson
---

Before writing any code, don't forget to include the header:

```c++
#include <ArduinoJson.h>
```

## Example

Here is an example to generate `{"sensor":"gps","time":1351824120,"data":[48.756080,2.302038]}`

```c++
//
// Step 1: Reserve memory space
//
StaticJsonBuffer<200> jsonBuffer;

//
// Step 2: Build object tree in memory
//
JsonObject& root = jsonBuffer.createObject();
root["sensor"] = "gps";
root["time"] = 1351824120;

JsonArray& data = root.createNestedArray("data");
data.add(48.756080);
data.add(2.302038);

//
// Step 3: Generate the JSON string
//
root.printTo(Serial);
```

## Step 1: Reserve memory space

ArduinoJson uses a preallocated memory pool to store the object tree; this is done by the `StaticJsonBuffer`.

In the case of a `StaticJsonBuffer`, the memory is reserved on the stack. The template parameter (`200` in the example) is the number of bytes to reserved.

Alternatively, you can use a `DynamicJsonBuffer` that allocates memory on the heap and grow as required. It is the preferred way for devices with a significant amount of RAM, like the ESP8266.

See also:

* [ArduinoJson memory model]({{ site.baseurl }}/doc/memory/)
* [FAQ: What are the differences between StaticJsonBuffer and DynamicJsonBuffer?]({{ site.baseurl }}/faq/what-are-the-differences-between-staticjsonbuffer-and-dynamicjsonbuffer)
* [FAQ: How to determine the buffer size?]({{ site.baseurl }}/faq/how-to-determine-the-buffer-size)

## Step 2: Build object tree in memory

Once the `JsonBuffer` is ready, you can use it to build your in-memory representation of the JSON string.

#### Arrays

You create an array like this:

```c++
JsonArray& array = jsonBuffer.createArray();
```

Don't forget the `&` after `JsonArray`; it needs to be a reference to the array.

Then you can add strings, integer, booleans, etc:

```c++
array.add("bazinga!");
array.add(42);
array.add(true);
```

You can add a nested array or object if you have a reference to it.
Or simpler, you can create nested array or nested objects from the array:

```c++
JsonArray&  nestedArray  = array.createNestedArray();
JsonObject& nestedObject = array.createNestedObject();
```

#### Objects

You create an object like this:

```c++
JsonObject& object = jsonBuffer.createObject();
```

Again, don't forget the `&` after `JsonObject`, it needs to be a reference to the object.

Then you can add strings, integer, booleans, etc:

```c++
object["key1"] = "bazinga!";
object["key2"] = 42;
object["key3"] = true;
object["key4"] = 3.1415;
```

You can add a nested array or object if you have a reference to it.
Or simpler, you can create nested array or nested objects from the object:

```c++
JsonArray&  nestedArray  = object.createNestedArray("key5");
JsonObject& nestedObject = object.createNestedObject("key6");
```

> ##### Other JsonObject functions
> * `object.set(key, value)` is a synonym for `object[key] = value`
> * `object.containsKey(key)` returns `true` is the `key` is present in `object`
> * `object.remove(key)` removes the `value` associated with `key`
{: .alert .alert-info }

## Step 3: Generate the JSON string

There are two ways tho get the resulting JSON string.

Depending on your project, you may need to dump the string in a classic `char[]` or send it to a `Print` implementation like `Serial` or `EthernetClient`.

Both ways are the easy way :-)

#### Use a classic `char[]`

Whether you have a `JsonArray&` or a `JsonObject&`, simply call `printTo()` with the destination buffer, like so:

```c++
char buffer[256];
array.printTo(buffer, sizeof(buffer));
```

> ##### Want an indented output?
> By default the generated JSON is as small as possible. It contains no extra space, nor line break.
> But if you want an indented, more readable output, you can.
> Simply call `prettyPrintTo` instead of `printTo()`:
> 
> ```c++
> array.prettyPrintTo(buffer, sizeof(buffer));
> ```
{: .alert .alert-info }

#### Send to a `Print` implementation

It is very likely that the generated JSON ends up in a stream like `Serial` or `EthernetClient `, so you can save some time and memory by doing this:

```c++
array.printTo(Serial);
```

And, of course if you need an indented JSON string:

```c++
array.prettyPrintTo(Serial);
```

> ##### About the Print interface
> The library is designed to send the JSON string to an implementation of the `Print` interface that is part of Arduino.
> In the example above we used `Serial`, but they are many other implementations that would work as well, including: `HardwareSerial`,  `SoftwareSerial`, `LiquidCrystal`, `EthernetClient`, `WiFiClient`, `Wire`...
> When you use this library out of the Arduino environment, it uses its own implementation of `Print` and everything is the same.
{: .alert .alert-info }

#### Length of the output data

If you need to know the length of the output data beforehand, use the `measureLength()` method:

```c++
int len = array.measureLength();
```

That comes in handy when you need to calculate the `Content-Length` when posting JSON data over HTTP.

## Where to go next?

<a href="https://leanpub.com/arduinojson/"><img src="{{site.baseurl}}/images/cover200.png" class="float-right" alt="Mastering ArduinoJson"></a>

In the [ArduinoJson ebook](https://leanpub.com/arduinojson/), there is a step-by-step tutorial to learn how to serialize JSON with the library.

The book contains a brand new tutorial on JSON serialization.

If you're not familiar with C++ references, the book also contains a quick C++ course to catch up with those things. For example, this chapter also explains the differences between a `char[]`, a `char*` or a `String`.