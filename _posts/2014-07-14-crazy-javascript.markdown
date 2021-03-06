---
layout: post
title:  "JavaScript: pain found with == vs ===."
date:   2014-07-14 14:37:25
categories: ['javascript']
---

This is an interesting list of things that you may want to be aware of as you write code as a JavaScript developer.

Most of my difficulty with this language has involved `==` and if evaluation in server-side javascript when values may or may not be set.  If you're being clever with logic expressions, it may be tempting to convert `A || B || C` to FALSE if none are true, but really it's the last entry in the list (C).  Think of the last value as an "ELSE" clause. 

In any case, consider using === whenever possible to reduce ambiguity.

### *==* vs *===*
{% highlight javascript %}
1 == 1;  //true
1 == "1"; //true - WHAT??
1 === 1; //true
1 === "1"; //false
1 == true; //true....WTF!
"" == false; //true...!!!!!
"" === false; //false
0 == false; //true
9 == true; //false
9 == false; //false
undefined == undefined; //true
undefined === undefined; //true
undefined == null; //true
undefined === null; //false
{% endhighlight %}

### Logic Expression Weirdness

It's important to remember that logic expressions like these are not actually booleans.  In an OR expression, the first not-evaluated-to-false expression is returned.  IF THERE ARE NONE - IT RETURNS THE LAST ENTRY REGARDLESS.  It is easy to screw yourself here. 

{% highlight javascript %}
true || false; //true
false || false || true; //true...so far so good
false || "1"; //"1"
false || 1 || true; //1
false || 0; //0
0 || false; //false
(0 || false || "0"); //"0"
(0 || false || "0")==0; //true
false || "false" || true; //"false"
false || "" || 0 || "0" || 1; //"0"
false || 0 || ""; //""
{% endhighlight %}

### If evaluation

`If` succeeds if the expression evaluates to any string other than "", true, or 1.
`If` fails if the expression evalutes to anything other than the above.

{% highlight javascript %}
//if(x) is not the same as if(x==true)
var result = "true" || false; //"true"
if(result){
  Console.log("result"); //prints
}
if(result == true){
  Console.log("result == true"); //DOES NOT PRINT
}

//last false wins
var falseOrZero = false || 0; //0
var zeroOrFalse = 0 || false; //false

if(falseOrZero || zeroOrFalse){
  Console.log("does not print"); //does not print
}
if(falseOrZero == false){
  Console.log("prints"); //prints
}
if(zeroOrFalse == false){
  Console.log("prints"); //prints
}

//"true" isn't true?
var trueStringOrOne = "true" || 1;
var oneOrTrueString = 1 || "true";
if(trueStringOrOne){
  Console.log("prints"); //prints
}
if(oneOrTrueString){
  Console.log("prints"); //prints
}
if(oneOrTrueString == true){
  Console.log("prints"); //prints
}
if(oneOrTrueString == trueStringOrOne){
  Console.log("prints"); //prints
}
if(trueStringOrOne == true){
  Console.log("does not print"); //does not print
}
{% endhighlight %}

As I run across more, I'll keep adding them to this list.

The moral of this story is to not use `==`.  Please use `===`.  

Also, some will write `if(true==condition)` for readability in lieu of `if(condition)`. If you do this in conjunction with `==`, and especially if others work on the same codebase, expect to be screwed repeatedly.

