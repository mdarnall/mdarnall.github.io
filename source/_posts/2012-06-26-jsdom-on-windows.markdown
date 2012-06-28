---
layout: post
title: "JSDom on Windows"
date: 2012-06-26 15:41
comments: true
categories: JavaScript 
published: true
---

[JSDom](https://github.com/tmpvar/jsdom) is a JavaScript implementation of the W3C DOM on Node.js. For our use case it proved to be a good solution to running unit tests that
required the DOM. 

A simple `npm install jsdom` installed jsdom just fine on \*nix
systems but we needed it working on both \*nix and Windows. Installing it on Windows required a few
additional prerequisites. At the time several google searches unturned
good tips, but none of them worked fully for us. This is an attempt at a
complete picture of the issues and solutions.The issues discussed here may change as node progresses as it has
changed several times in the recent past. 

The main issue at time of writing is that jsdom in turn requires a node module called [contextify](https://github.com/brianmcd/contextify/). Contextify
requires a C++ addon, which must be built for the given platform. The
way node builds these native addons is the build tool [node-gyp](https://github.com/TooTallNate/node-gyp/).
 
### Prerequisites
* Node.js (at writing time using 0.6.19) and NPM (1.1.24). These are
   packaged together when using the [Windows installer](http://nodejs.org/#download) 
  * Note: That my initial tests using node 0.8.0 and npm (1.1.32) also
    worked the same.
* Python: Node-gyp currently recommends 2.7.x   
  * Add python to your PATH 
  * Add a new environment variable `PYTHON=C:\path_to_python\python.exe` 
    * This wasn't listed in the node-gyp instructions but based on the output of the script it seemed necessary. 
* Microsoft Visual C++. The express version works as a free
   alternative. 

### Installing
Once these prerequisites are installed jsdom is installed just like any
other module via:

```
npm install jsdom
``` 

You should see output from node-gyp and then msbuild being invoked to compile the native module. In our experience
msbuild would output a warning that was safe to ignore: 

```
MSB8012: TargetExt (.dll) does not match the Linker's OutputFile property value (.node)
```

One of the other issues we had was finding a good, simple example to do a sanity check 
to make sure things were installed correctly. We found that the best example seemed to be the one from the site that
loaded JQuery via a CDN and listed the links from Hacker News. 
{% gist 3007798 %}

I'm working on a related blog post about our JavaScript testing strategy
and how we are using tools like Mocha, JSDom and others to write a
test-driven web application. 
