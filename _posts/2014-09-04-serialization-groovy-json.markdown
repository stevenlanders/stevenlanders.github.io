---
layout: post
title:  "Groovy: Writing objects without Serializable"
date:   2014-09-04 00:00:00
categories: ['groovy','java','serialization']
---

### Problem Summary

You want to serialize an object to a file, but it doesn't implement [Serializable](http://docs.oracle.com/javase/7/docs/api/java/io/Serializable.html).  

{% highlight groovy %}
class User{
    String name;
    String phone;
    String favoriteColor;
}
{% endhighlight %}

The Serializable interface is annoying.  It's a basic object.  Just let me read/write the object!!


### JSON Proposal

One solution is very straightforward - don't use proper "Serialization" - because it may not actually be what you need.  The downside to this is that `==` is not satisfied, but you probably do not need this!

Here's a basic example using Groovy's helpful [JsonOutput](http://groovy.codehaus.org/gapi/groovy/json/JsonOutput.html) and [JsonSlurper](http://groovy.codehaus.org/gapi/groovy/json/JsonSlurper.html) classes.

{% highlight groovy %}
def testUser = new User(
    name: 'steven',
    phone: '1-770-555-1212',
    favoriteColor: 'red'
)

//convert to json
def testUserJson = JsonOutput.toJson(testUser)

//write to file
def outputFile = new File("testUser.json")
outputFile.write(testUserJson)

def testUser2 = JsonSlurper.parse(outputFile) as User
{% endhighlight %}


### Cache implemenation using this approach

I added an implementation of the Grails Cache plugin that uses this approach here: [Grails Filesystem Cache Plugin](https://github.com/stevenlanders/grails-plugin-cache-filesystem)

This had the added complexity of dyanmic filesnames.  Given arbitrary key types, and filename length limitations - I used *SHA256(json(key)).json* for the filename.


