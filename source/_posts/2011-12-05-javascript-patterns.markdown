---
layout: post
title: "JavaScript Patterns"
date: 2011-12-05 12:03
comments: false
categories: 
---

This is an ongoing set of notes based on my learning of JavaScript
patterns and best practices. It's a collection of knowledge from various
sources. 

#####Global Variables
It's a good idea to minimize the number of global variables in a
javascript application. The main reason is because of naming collisions
between code bases. If two seperate code bases declare global variables
with the same name unintended consequences are often a result.  

Two main features of javascript as a language
make the issue easier to create:  

- Not having to declare variables before using them  
- Implied globals - any variable you don't declare becomes a property
   of the global object  
    - ES5 strict mode will throw an error if assignments are made to implied globals  

The easiest way to avoid global variables is to always declare variables
with the `var` keyword. 

##### Object Literals 
_Object Literal Notation_ is ideal for on-deman object creation. You can
start with a blank object and add functionality as you need.  

    var dog = {};
    // adding a property 
    dog.name = 'benji';
    // add a method
    dog.getName = function(){ 
      return dog.name;
    };

But you can also create the same object at once:  

    var dog = {
      name : 'benji',
      getName : function () {
        return this.name;
      }
    };

I like this quote from the JavaScript Patterns book
>Another reason why the literal is the preferred pattern for object crea- tion is that it emphasizes that objects are simply mutable hashes and not something that needs to be baked from a “recipe” (from a class).

##### Custom Constructor Functions
An alternative to object literals for creating custom objects is to use
_Custom Constructor Functions_. 

{% include_code patterns.js/src/Person.js %}

Constructors are function that are invoked with `new`. When `new` is not
used `this` inside the constructor will refer to the global object
(`window` in a browser) instead of the object instance. So a helpful
pattern is to enforce the use of `new` with a self-invoking contructor

    var Person = function(name) { 
      if(!(this instanceof arguments.callee)){
        return new arguments.callee();
      }
      this.name = name;
      this.say = function () {
        return "My name is " + this.name;
      };
    }

