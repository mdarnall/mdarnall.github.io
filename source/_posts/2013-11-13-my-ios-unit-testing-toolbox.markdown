---
layout: post
title: My iOS Unit Testing Toolbox
date: 2013-11-13 18:15
comments: false
categories: ios
---

For the past 6 months I've been doing iOS development for [Salesforce](http://salesforce.com). Having spent most of my previous time doing JavaScript, Ruby and .NET development I'd gotten used to certain tools to accomplish my daily workflow. This is a quick writeup on the tools I'm currently using to get a similar experience in iOS.


### Testing Frameworks

I'm big on unit testing - I mean I love it, and I love TDD. So naturally the tools I use need to support that workflow. I've come to really love BDD style unit testing frameworks in [Ruby with RSpec](https://github.com/rspec/rspec) and [Mocha](http://visionmedia.github.io/mocha/) for JavaScript - so this was the first tool I went looking for.

In the Objective-C community there are several frameworks, that at the surface, offered the BDD style I was looking for: 

* [Kiwi](https://github.com/allending/Kiwi) - "Simple BDD for iOS"
* [Cedar](https://github.com/pivotal/cedar) - "BDD style testing for Objective-C"
* [Specta](https://github.com/specta/specta) - "A light-weight TDD / BDD framework for Objective-C & Cocoa"

All three have similar features and support the style you would expect. Here is an example of using Specta to describe an objects behavior:

``` obj-c
import "specta.h"

SpecBegin(Thing)

describe(@"Thing", ^{
  
  beforeAll(^{
    // This is run once and only once before all of the examples
    // in this group and before any beforeEach blocks.
  });

  beforeEach(^{
    // This is run before each example.
  });

  it(@"should do stuff", ^{
    // This is an example block. Place your assertions here.
  });

  it(@"should do some stuff asynchronously", ^AsyncBlock {
    // Async example blocks need to invoke done() callback.
    done();
  });

  
  describe(@"Nested examples", ^{
    it(@"should do even more stuff", ^{
      // ...
    });
  });

  pending(@"pending example");
  
  afterEach(^{
    // This is run after each example.
  });

  afterAll(^{
    // This is run once and only once after all of the examples
    // in this group and after any afterEach blocks.
  });
});

SpecEnd

```

After playing with each of the three I decided to go with Specta and have been really happy with that choice. My criteria was based on:

* The ability to easily add the framework to an existing XCode project.
* The ability to write asynchronous test cases. Often I'd need to test objects that take block arguments for callbacks - this was a must. 
* Integration into other mainstream tools. XCTool and XCode are well supported with Specta. 
* A good project "pulse" on [GitHub](https://github.com/specta/specta/pulse/monthly). 


### Matcher Library

Along with a good BDD style testing framework I wanted to find a matcher library that gave me a more fluent, readable syntax similar to that of RSpec or [Chai](http://chaijs.com/) in JavaScript. Something like:

``` obj-c
expect(@"foo").to.equal(@"foo"); 
expect(foo).notTo.equal(1);
expect([bar isBar]).to.equal(YES);
expect(baz).to.equal(3.14159);

```


The Specta project came to the rescue again with [Expecta](https://github.com/specta/expecta). It has proven to work well and provides a lot of features:

* A good set of built-in [matchers](https://github.com/specta/expecta#built-in-matchers)
* The ability to define your own matchers
* Dynamic predicate matchers. Being able to write something like `expect(lightSwitch).isTurnedOn();`  with minimal effort is great.
* Testing framework agnostic. Expecta will work with not just Specta but other testing frameworks.
* A good project "pulse" on [GitHub](https://github.com/specta/expecta/pulse/monthly)


### Isolation Library

In unit testing I often make use of an [Isolation Library](http://osherove.com/blog/2008/9/20/goodbye-mocks-farewell-stubs.html). While these are often referred to as Mock libraries, it's really just a library for making fake objects on the fly. I primarily need Mock and Stub objects and so far I've been using [OCMock](http://ocmock.org) with good success.

Often with fake objects you want to create Stub objects that are not "strict" - meaning they don't complain if methods you didn't explicitly setup are called. One bummer is that with OCMock you must remember to create a "Nice Mock" in order to get that behavior:

``` obj-c
id mock = [OCMockObject niceMockForClass:[SomeClass class]]

```

Argument constraints are another important feature for Mock objects. OCMock has a decent set of built-in argument constraints, and when the built-in constraints won't cut it you can reach for a custom predicate constraint via a block you implement:

``` obj-c
[[mock expect] someMethod:[OCMArg checkWithBlock:^BOOL(id value) { 
    /* return YES if value is ok */ 
}]];

```

This actually turns out to be the way you can mock calls of methods that take block arguments. In a recent project we had an API Client object that used [AFNetworking](https://github.com/AFNetworking/AFNetworking) to implement it's HTTP communication. Each method call would take a callback for when that communication completed. For unit tests that needed to Stub or Mock this interaction I used the following approach:

``` obj-c
// stub out the call to doAPIWork and callback with an error condition
[[client stub] doAPIWork callback:[OCMArg checkWithBlock:^BOOL(id param){
    // cast the param to our block type
    void (^passedBlock)(NSArray *, NSError *) = param;
    // call the passed block back with our stubbed response
    passedBlock(nil, [NSError errorWithDomain:@"" code:500 userInfo:@{}]);
    // indicate that the argument was fine to OCMock
    return YES;
}]];
            
```

That being said - I'm would encourage others to also evaluate [OCMockito](https://github.com/jonreid/OCMockito) from [Jon Reid](http://qualitycoding.org/). Some key differences are:


* Mock objects are always "nice," recording their calls instead of throwing exceptions about unspecified invocations. This makes tests less fragile.

* No expect-run-verify, making tests more readable. Mock objects record their calls, then you verify the methods you want.

* Verification failures are reported as unit test failures, identifying specific lines instead of throwing exceptions. This makes it easier to identify failures.


### Test Runner

Writing tests is important, but equally important is running those tests to get feedback and to quickly iterate during development. For running tests I've been making heavy use of [XCTool](https://github.com/facebook/xctool). XCTool is a tool for building and testing your application at the command line. This was big not only for those who prefer that local development environment but for running the tests during Continuous Integration on a remote server.

One thing I quickly added to our project was XCTool's ability to have default arguments passed to the command by having a `.xctool-args` file in the root of your project. 

``` json
[
  "-workspace", "YourWorkspace.xcworkspace",
  "-scheme", "YourScheme",
  "-configuration", "Debug",
  "-sdk", "iphonesimulator",
  "-arch", "i386"
]
```

With that you can easily run your tests by running `$xctool test`

Better yet - for those who like to practice TDD you can combine XCTool with the power of [Guard](https://github.com/guard/guard) so that you tests will automatically run when you make changes to any of your project source files. An easy way to get started with this is to use the [guar-xctool-test gem](https://github.com/siuying/guard-xctool-test) from Francis Chong.

### AppCode

I've been using [AppCode](http://www.jetbrains.com/objc/) over XCode for a little while now. This was mainly because I had become so used to the features of JetBrains IDE's over the years it was hard to go back. I've used their products for developing Java with IntelliJ, Ruby with RubyMine, JavaScript with WebStorm and .NET with ReSharper. Features like refactoring, code analysis and source navigation make it at least worth a try. 

***

_The tools you use are really a personal preference - These are mine. As I'm sure you already know these are not the only options out there. I encourage you to find the tools that make you productive and happy._
