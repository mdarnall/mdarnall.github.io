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

#### Global Variables
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

#### Objects 
Objects are mutable keyed collections that contain properties. A
property can be any JavaScript value except for `undefined`.

_Object Literal Notation_ is ideal for on-deman object creation. You can
start with a blank object and add functionality as you need.  
``` javascript
    var dog = {};
    // adding a property 
    dog.name = 'benji';
    // add a method
    dog.getName = function(){ 
      return dog.name;
    };
```
But you can also create the same object at once:  
``` javascript
    var dog = {
      name : 'benji',
      getName : function () {
        return this.name;
      }
    };
```
I like this quote from the JavaScript Patterns book
>Another reason why the literal is the preferred pattern for object creation is that it emphasizes that objects are simply mutable hashes and not something that needs to be baked from a “recipe” (from a class).

##### Prototype
JavaScript objects are all linked to a _prototype_ object where it can
inherit properties. This is important for code-reuse patterns discussed
later. Object literals are linked to the `Object.prototype` by default. 

### Functions
* Functions are first class objects. They can be passed around as values
  or augmented with properties and methods
* Provide local scope. Declarations of local variables get _hoisted_ to
  the top of local scope.  

__Syntax for creating functions__  


Named function expressions

``` javascript
var add = function add (a,b) { 
  return a+b;
}
```
Anonymous functions. Same as above but without a name:
``` javascript
var add = function (a,b) { 
  return a + b;
}
```
Function Declarations:
``` javascript
function add (a,b) { 
  return a + b;
}
```
__Invocation__

When a function is invoked it is passed it's declared parameters and two
additional ones:  

* a reference to `this`
* a reference to `arguments`

The reference to `this` depends on how the function was invoked.  


__Method Invocation__
When a function is a property of an object, it is refered to as a
method. When a method is invoked `this` refers to the containing object. 

```javascript
  var counter = {
    count : 0, 
    increment : function () {
      this.count += 1;
    }
  };

  counter.increment();
```

__Function Invocation__
When a function is not a property of an object, the function's reference
to `this` is bound to the global object. 

```javascript
  function add (a, b) { 
    return a + b; // 'this' refers to the global object here
  }
```
__Contructor Invocation__
When an object is created with the `new` keyword it's refered to as a
_Contructor_. The object's reference to `this` is bound to that object. 

```javascript
  var MyObj = function (){
    this.name = 'Matt';
  }

  var obj = new MyObj();
  obj.name // 'Matt'
```

When `new` is not used `this` inside the constructor will refer to the global object instead of the object itself. So a helpful pattern is to enforce the use of `new` with a _self-invoking contructor_  

``` javascript
    var Person = function(name) { 
      if(!(this instanceof arguments.callee)){
        return new arguments.callee();
      }
      this.name = name;
      this.say = function () {
        return "My name is " + this.name;
      };
    }
```

##### Scope 
Scope is determined by functions, not by blocks in JavaScript.
Parameters and variables defined in a function are not visible outside
of that function. Also variables declared inside a function are visible
anywhere everywhere within a function. One interesting case is when an
inner function has a longer lifetime than its outer function:

__Immediate Functions__   
A pattern that wraps a function and immediately executes it. It helps
avoid poluting the global namespace and also creates a closure,
protecting _private_ variables. 

```javascript
  var counter = (function(){
    var count = 0;
    return {
      increment : function (){
        count += 1;
      }, 
      getCount : function (){
        return count;
      }
    };
  })();
  counter.increment();
  counter.getCount(); // 1
  typeof counter.count; // undefined

```
