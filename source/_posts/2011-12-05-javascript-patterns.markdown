---
layout: post
title: "JavaScript Patterns"
date: 2010-12-05 12:03
comments: false
categories: JavaScript 
---

This is an ongoing set of notes based on my learning of JavaScript
patterns and best practices. It's a collection of knowledge from various
sources. 

In addition code examples are being maintained as an executable set of
specifications in the [patterns.js](https://github.com/mdarnall/patterns.js) repo on GitHub. 


### Objects 
Objects are mutable keyed collections that contain properties. A
property can be any JavaScript value except for `undefined`.

_Object Literal Notation_ is ideal for on-demand object creation. You can
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

###### Syntax for creating functions  

Named function expressions

``` javascript
var add = function add (a,b) { 
  return a + b;
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
###### Invocation

When a function is invoked it's  passed the declared parameters and two
additional ones:  

* a reference to `this`
* a reference to `arguments`

The reference to `this` depends on how the function was invoked.  


###### Method Invocation
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

###### Function Invocation
When a function is not a property of an object, the function's reference
to `this` is bound to the global object. 

```javascript
  function add (a, b) { 
    return a + b; // 'this' refers to the global object here
  }
```
###### Constructor Invocation
When an object is created with the `new` keyword it's refered to as a
_Constructor_. The object's reference to `this` is bound to that object. 

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

###### Scope 
Scope is determined by functions, not by blocks in JavaScript.
Parameters and variables defined in a function are not visible outside
of that function. Also, variables declared inside a function are visible
anywhere within it -  One interesting case is when an
inner function has a longer lifetime than its outer function.

###### Immediate Functions   
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

### Code-Reuse and Inheritance 

Reusing code is an important topic to any discussion of software
development. In classical languages this is usually done with
inheritance. JavaScript supports many different ways in which code can
be reused. I like this quote from JavaScript: The Good Parts when
thinking about how JavaScript differs from other languages:

> In classical languages, objects are instances of classes, and a class
> can inherit from another class. JavaScript is a _prototypal_ language,
> which means that _objects_ inherit directly from other _objects_

The most natural inheritance pattern is to embrace the prototypal behavior 
and focus on objects inheriting properties of other objects. 

Prototypal inheritance is easy with the `Object.create` method in
ECMAScript 5:

```javascript
var parent = {
  name : 'Daddy'
};
var child = Object.create(parent);
child.name // 'daddy'
```

This method is easy to pollyfil in environments that don't support it
natively:

```javascript
if (!Object.create) {
    Object.create = function (o) {
        if (arguments.length > 1) {
            throw new Error('Object.create implementation only accepts the first parameter.');
        }
        function F() {}
        F.prototype = o;
        return new F();
    };
}
```
Another approach to code-reuse to the apply _psuedoclassical_ patterns
of inheritance to JavaScript. The most straight forward and versitile way
is called the _Proxy Constructor Pattern_. The idea is to have the child 
prototype point at a _proxy_ object that in turn is linked to the parent 
via it's prototype. 

``` javascript
  var inherit = (function(){
    var F = function (){};
    return function (C,P){
      F.prototype = P.prototype;
      C.prototype = new F();
      C.parent = P.prototype;
      C.prototype.constructor = C;
    };
  })();

inherit(Child, Parent);
```

It is possible to make this pattern a little easier to use by wrapping
it in some syntactical sugar, in a pattern called _Klass_  

```javascript

  var Klass = function(Parent, props) {
    var Child
    , F
    , i;

    // create a constructor function
    Child = function (){
      if(Child.parent && Child.parent.hasOwnProperty('initialize')){
        Child.parent.initialize.apply(this, arguments);
      }
      if(Child.prototype.hasOwnProperty('initialize')){
        Child.prototype.initialize.apply(this, arguments);
      }
    };

    // inherit via the proxy prototype pattern
    Parent = Parent || Object;

    F = function (){};
    F.prototype = Parent.prototype;
    Child.prototype = new F();
    Child.parent = Parent.prototype;
    Child.prototype.constructor = Child;

    // copy properties
    for(i in props){
      if(props.hasOwnProperty(i)){
        Child.prototype[i] = props[i];
      }
    }

    return Child;
  };

```
It can then be used like: 

```javascript
    var Man = Klass(null, {
        initialize : function (name){
          this.name = name;
        }, 
        getName : function (){
          return this.name;
        }
    });

    var SuperHuman = Klass(Man, {
      initialize : function (){}, 
      getName : function (){
        var name = SuperHuman.parent.getName.call(this);
        return "I am " + name;
      }
    });

```

Another pattern in code-reuse is the concept of borrowing methods. In
cases where it doesn't make sense to inherit all of the properties you
can just borrow the ones you need:

```javascript

notmyobj.dostuff.apply(myobj, [params]);

```
### Global Variables
It's a good idea to minimize the number of global variables in a
JavaScript application. The main reason is because of naming collisions
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
