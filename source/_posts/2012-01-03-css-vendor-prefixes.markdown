---
layout: post
title: CSS Vendor Prefixes
date: 2011-03-04
comments: true
categories: css
---
A quick note on using vendor-specific properties: when doing so it is
generally a good idea to include the non-prefixed property as well, after all of
the prefixed versions. 

This will ensure that when the time comes and the browser supports the
property entirely it will be used. It will also override the prefixed version when that time comes. 

An example with border radius: 
```css
.myClass { 
  -moz-border-radius : 20px;
  -webkit-border-radius: 20px;
  border-radius: 20px;
}
```
Or even better using a pre-compilation tool like [Compass](http://compass-style.org/reference/compass/css3/border_radius/) you can just do something like: 
```css
.myClass { 
  @include border-radius(4px);
}
```
And based on how you have configured the supported browsers it will take
care of emitting the vendor specific properties. 
