---
layout: post
title: "Running JavaScript Tests Like a Boss"
date: 2012-08-18 15:00
comments: true
categories: JavaScript 
published: true
---

One of the things I've always been driven to in software development is having a
codebase that is easily tested and where test execution is easy and
integrated well into the development workflow. As the majority of my time has been
spent writing JavaScript and less server-side code I've been trying
different approaches to writing testable JavaScript code and
incorporating the same proven practices of continuous integration and
test-driven development into my workflow. 

The goal of this post is to discuss the various
options that are out there for easily running and incorporating your
JavaScript suite into the development workflow. A lot of discussion in the community has taken place in terms on improving the toolchain of developing JavaScript and building and
running a solid test suite is an integral piece to that. 

### Browser-Based Test Runner
Most frameworks have html based runners. These are fine to get
started as they provide a good way to run your test suite in a browser
of your choice. As we will
see later other options will use this as the main method of running the
suite in an automated fashion.

Let's look at some examples. All popular testing frameworks include a browser-based 
test runner:  

* [QUnit](http://qunitjs.com/)
* [Jasmine](http://pivotal.github.com/jasmine/)
* [Mocha](http://visionmedia.github.com/mocha/)
* [Buster.JS](http://busterjs.org/docs/browser-testing/)

##### Pros 
* Easy - This is by far the easiest option to get started with. Simply
  use the included html runner in your test framework and visit the page
  in your browser of choice. 
* Representative - This option does a good job at proving your tests are
  reflective of an actual browser environment. It may only represent one
  of the browser environments you may wish to target, but we'll explore
  other options to help with this issue. 

##### Cons
* Slower - In comparison to other options it can be slower to load up a
  browser and visit the runner page. 
* Harder to automate - While it may be possible to automate the process
  of running the suite through a real browser it's not as easy as some
  of the other approaches.
* Requires the developer to host and to open the runner in the browser,
  instead of a simple command or something built into your build process. 

### Headless Browser-Based Test Runner
This is a variation of our first option. The idea is to use a "headless" browser to
load and parse the browser-based test runner. The difference is that you
don't incur the cost of loading an entire browser, but it means that you
must use a script to parse and display the results of the browser-based
runner. 

Several headless browsers are available: 

* [PhantomJS](http://phantomjs.org/) - Headless Webkit with a JavaScript
  API based on [QTWebKit](http://trac.webkit.org/wiki/QtWebKit).   
* [Zombie.js](http://zombie.labnotes.org/) - Insanely fast, headless full-stack 
  testing using Node.js. 

This seems to be the option that is gaining the most popularity amongst developers. Several
projects are using this approach:

* [Twitter Bootstrap](https://github.com/twitter/bootstrap/) - Their plugin unit-tests use PhantomJS to run.
* [Grunt](https://github.com/cowboy/grunt) - The task-based build tool for JavaScript has a built-in 
  QUnit task that will run your tests with PhantomJS.
* [Soundcloud Mobile](http://backstage.soundcloud.com/2011/09/mobile-unit-testing/) - Used QUnit and PhantomJS on the mobile
  project. 

##### Pros
* Fast - Spinning up a headless browser is generally faster than using
  a fully fledged browser. 
* Easy automated - The process of spinning up the headless browser
  can easily be scripted to run as part of your local development cycle
  or as part of your continuous integration cycle on a remote machine. 
  
##### Cons
* The parsing code must be written to transform the browser runner to
  the command line output. This component may already exist based on the
  test framework you are using. For example, [QUnit includes a runner for PhantomJS](https://github.com/jquery/qunit/tree/master/addons/phantomjs).
  Any features offered by the framework must be extracted from the
  html output of the browser-based runner. For example, timings or
  stack traces for failures. Again, this may not be an issue unless you
  are writing the parsing code.
* Not represntative - The validity of the results really only indicates that your tests pass
  on the JavaScript engine of the headless browser, not a browser that
  your end users would use. As we'll see there are other approaches that mitigate this downside. 

### DOM Emulation in Node.js
This approach is an interesting one. While some testing frameworks
already support testing JavaScript running under node.js you can
incorporate a W3C DOM implementation into node so that tests cases that
need a DOM can run along side those that don't. This approach isn't talked about as much in the
community but I find it to be a nice hybrid approach for testing locally
and as a first pass test with continuous integration. 

This approach needs the following:

* [JSDom](https://github.com/tmpvar/jsdom/)
* A testing framework that supports node, like:  
  * [Mocha](http://visionmedia.github.com/mocha/)  
  * [Buster.js](http://busterjs.org/docs/node-testing/)  
* A bootstrapping script that imports the right node modules and test
  suites.

##### Pros
* Fast - Similar to the "Headless" testing approach these tests run
  fast.
* Native - No reason to write and maintain parsing code that extracts the testing
  frameworks output. Instead use a testing framework that natively
  supports running tests under node.js and add DOM support where needed. 
* Easily automated - Just like the "Headless" testing approach node.js
  is easily integrated into normal developer workflow via the command
  line or on a server running your continuous integration process. 

##### Cons
* Not representative - Again, like the "headless" approach these tests
  really only validate your suite against JSDom's W3C DOM implementation - not
  against any real browsers that your end users may use.  

### Browser Automation
Another good approach is to automate one or more real browsers that 
run your test framework's built-in runner. This is conceptually
similar to the first approach, except you use the help of a tool to automate 
running the test in one or more browsers. This approach can be run locally in your development workflow, or
using a cloud-based service. There are several interesting projects 
that help you with this approach:  

* [Yeti](http://yeti.cx/) - A command-line tool for launching JavaScript
  unit tests in one or more browsers and reporting results. Currently
  only supports tests written in YUI Test. 
* [Bunyip](http://ryanseddon.github.com/bunyip/) - An effort to combine
  BrowserStack and Yeti for cross browser testing as well as add support for other testing frameworks like Jasmine.
* [Testacular](http://vojtajina.github.com/testacular/) - Another
  command-line tool for launching JavaScript tests in one or more
  browsers as well as headless browsers. Relatively new at time of
  writing, but it has some interesting features. 
* [Test Swarm](https://github.com/jquery/testswarm/wiki) - Distributed
  integration testing for JavaScript. [It is limited in testing framework support](https://github.com/jquery/testswarm/tree/master/scripts/addjob#test-suite-runs)
* Cloud Based Services:
  * [BrowserStack](http://www.browserstack.com/automated-browser-testing-api) - Provides browser-as-a-service for automated cross-browser testing. Because it is just an API for spinning up new browsers it is usually combined with another tool like Bunyip that needs browser instances to work. In fact, the JQuery project uses the combination of Test Swarm and BrowserStack to do it's cross-browser testing.  
  * [Testling](http://testling.com/) - A cross-browser testing platform
    that has an web API to allow running your tests in the cloud
on various real browsers as well as locally using JSDom. Testling provides an
integrated testing framework, but [adapters to other frameworks can be created as well.](http://substack.net/posts/1db3bb/roll-your-own-test-runner-for-testling)


##### Pros
* Representative - Yours tests are run in a variety of real browsers, potentially event against a variety of devices, giving you more confidence in your test results. Unfortunately nothing can truly replace the experience of running actual code in a real browser.
* Flexible - Some of the tools provide a way to run your tests locally
  using something like JSDom or PhantomJS, but then also in the cloud
against real browsers. 

##### Cons
* Complexity - Because of the fact that you are automating one or more
  browsers which have anomalies and can sometimes be fragile the
overall solution is more complex than some of the other approaches.
* Setup Time - Depending on the library and tools used it may require
  more of an investment of time to get get up and running. 
* Cost - Cloud-based services are not free but are relatively
  inexpensive. 


While there are many options out there it really comes down to your
testing goals. Choosing a tool that is flexible and that can run in
multiple ways is the best choice as it allows you to adapt as your
needs change. 
