---
layout: post
title: "JavaScript Testing Like a Boss"
date: 2012-06-25 22:08
comments: true
categories: JavaScript 
published: false
---

_This is really only for browser-based apps_ 

Once you have your framework of choice and you have written a suite of 
tests what are you options for running those tests? This post will attempt to get into the options that are available and to discuss
some of the pros and cons of each approach. While I am not new to
testing JavaScript, I was dissapointed in the research I did to find out
more about the options when it came to running those tests. 

A lot of discussion in the community has taken place in terms on
improving the toolchain of developing JavaScript and building and
running a solid test suite is an integral piece to that. 
* Paul Irish spoke at Fluent Conf and JSConf 2012 on Tooling, 
* JSConf 2012 : several talks on Grunt, etc.

_Note: almost make a scoring system. Each option does better/worse at
several of the same points_

### Browser runner
All frameworks I've used have html based runners. These are fine to get
started as they provide a good way to write tests that rely on the
browser environment.

##### Pros 
* As we will see in other options this option does a good job at proving
  your tests run in a real environment - meaning your results reflect
the results of your code in an environment that a user would actually
use. Now of course this is only one of the potentially many environments
your users would have but we will see a strategy based on this method
that could help with this.

#### Cons
* Slower - the overhead of launching a browser
* Harder to automate
* Usually requires hosting the runner with an HTTP server.

### Headless browser runner
Use a headless browser to load the html based runner and inspect it's
results. 
* Examples: Taking the test suite and running them with the popular
  [PhantomJS](http://phantomjs.org/) project. 

##### Pros
* Faster than the just using the browser alone
* More automatable - run at the command line
##### Cons
* The "scraping" code must be written to transform the browser runner to
  the command line output. This component may already exist based on the
runner/test framework you use. For example, [PhantomJS has examples of
integration with QUnit and Jasmine on their wiki](http://code.google.com/p/phantomjs/wiki/TestFrameworkIntegration)
* Any features offered by the framework must be extracted from the
  html output
* The validity of the results really only indicates that your tests pass
  on the JavaScript engine of the Headless browser

### Completly Headless Solution
* Use JSDom and Node
_What are the main differences here with the Healess version?_

### Browsers as a service  
Using the build in browser runner but running the suite of tests against
a number of browser combinations. Usually automated using cloud-based
services. 

* Examples: There are a few tools out there that help you with this
  strategy:
1. [Test Swarm] (https://github.com/jquery/testswarm/wiki)
2. [BrowserStack](http://www.browserstack.com/automated-browser-testing-api)
3. [Testling](http://testling.com/)

###### Pros
* Runs yours tests in a variety of real browsers - giving you more
  confidence in your test results. Unfortunately nothing can truly replace the experience of running actual code in a real browser.
* With the help of tools this process is doable. 

##### Cons
* Less documented? 
* More time to setup?
