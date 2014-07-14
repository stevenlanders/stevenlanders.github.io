---
layout: post
title:  "Crazy JavaScript - crazy examples from a crazy language"
date:   2014-07-14 14:37:25
categories: ['javascript']
---

JavaScript is crazy.  That said, we're all using it - and we like it.  This is an interesting list of things that you may want to be aware of as you write code as a crazy person.

### *==* vs *===*
{% highlight javascript %}
1 == 1;  //true
1 == "1"; //true - WHAT??
1 === 1; //true
1 === "1"; //false
1 == true; //true....WTF!
0 == false; //true
9 == true; //false
9 == false; //false
undefined == undefined; //true
undefined === undefined; //true
undefined == null; //true
undefined === null; //false
{% endhighlight %}

### Logic Expression Weirdness

It's important to remember that logic expressions like these are not actually booleans.  It is easy to screw yourself here. 

{% highlight javascript %}
true || false; //true
false || false || true; //true...so far so good
false || "1"; //"1"
false || 1 || true; //1
(0 || false || "0"); //"0"
(0 || false || "0")==0; //true

{% endhighlight %}



