---
layout: post
title: "JavaScript Testing Like a Boss"
date: 2012-06-25 22:08
comments: true
categories: JavaScript 
published: false
---

_This is really only for browser-based apps_ 

One of the things I've always been driven to in software development is having a
codebase that is easily tested. As the majority of my time has been
spent writing JavaScript and less server-side code I've been trying
different approaches to writing testable client-side code and
incorporating the same proven practices of continuous integration and
test-driven development. The goal of this post is to discuss the various
options that are out there for easily running and incorporating your
client-side test suite into the developer workflow. 

A lot of discussion in the community has taken place in terms on
improving the toolchain of developing JavaScript and building and
running a solid test suite is an integral piece to that. 
* Paul Irish spoke at Fluent Conf and JSConf 2012 on Tooling, 
* JSConf 2012 : several talks on Grunt, etc.

_Note: almost make a scoring system. Each option does better/worse at
several of the same points_



Let's take a look at some of the options:

### Browser-Based Test Runner
All frameworks I've used have html based runners. These are fine to get
started as they provide a fine way to run your test suite. As we will
see later other options will use this as the main method of running the
suite in an automated fashion.

* Examples: Most popular testing frameworks come with a browser-based
  runner:
[QUnit](http://qunitjs.com/)
[Jasmine](http://pivotal.github.com/jasmine/)
[Mocha](http://visionmedia.github.com/mocha/)
[Buster.JS](http://busterjs.org/docs/browser-testing/)

##### Pros 
* Easy - This is by far the easiest option to get started with. Simply
  use the included html runner of your test framework and visit the page
  your browser of choice. 
* Accurate - This option does a good job at proving your tests are
  reflective of an actual browser environment. It may only represent one
  of the browser environments you may wish to target, but we'll explore
  other options to help with this issue. 

##### Cons
* Slower - In comparison to other options it can be slower to load up a
  browser and visit the runner page. 
* Harder to automate - While it may be possible to automate the process
  of running the suite through a real browser it's not as easy as some
  of the other options.
* Usually requires hosting the runner with an HTTP server.

### Headless Browser-Based Test Runner
This is a variation of our first option. The idea is to use a "headless" browser to
load and parse the browser-based test runner. The difference is that you
dont incur the cost of loading an entire browser. 

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
