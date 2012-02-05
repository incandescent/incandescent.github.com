---
layout: post
title: "Extending Jasmine Matchers"
date: 2011-07-03 01:55
comments: true
categories: 
---
<p><a href="http://pivotal.github.com/jasmine/">Jasmine</a> is a behavior-driven development framework for testing JavaScript from <a href="http://pivotallabs.com/">Pivotal Labs </a> (I believe <a href="http://agaskar.com/">Rajan Agaskar</a> was/is the original author). I've been using Jasmine for a while now to test my JavaScript code. Today I ran into interesting situation where I wanted to test events bound to jQuery element. Basically by writing code like this:</p>
``` javascript
$('#el').bind('click', function (e) {
  // handler code goes here
});
```
<p>I wanted to know if element <strong>$('#el')</strong> had any events attached to it. I looked around but couldn't find any predefined matchers in jasmine or <a href="https://github.com/velesin/jasmine-jquery">jasmine-jquery</a> to accomplish it. I realized that it's pretty easy to extend jasmine and define my own matchers (which is pretty cool). Here is how you can do it:</p>
<p>First create your new matchers:</p>
``` javascript
var eventMatchers = {
  toHaveEvent: function (event) {   
    var data = jQuery.data(this.actual.get(0));
    for (obj in data) {
      if (data.hasOwnProperty(obj)) {
        for (eventName in data[obj].events) {
          if (data[obj].events.hasOwnProperty(eventName)) {
            if (event === eventName) {
              return true;
            }
          }
        }
      }
    }
    return false;
  }
};
```
<p>Then in your spec add your matchers inside beforeEach:</p>
``` javascript
describe("Element", function () {
  beforeEach(function () {
    this.addMatchers(eventMatchers);
  });
});
```
<p>That's it! Now you can use your new matchers inside your spec. In this case I was able to use it like this:</p>
``` javascript
describe("Element", function () {
  beforeEach(function () {
    this.addMatchers(eventMatchers);
  });
  
  describe("when event attached", function () {
    it("should contain attached event", function () {
      setFixtures("&lt;div id='el'&gt;&lt;/div&gt;");
      $('#el').bind('click', function () {});
      expect($('#el')).toHaveEvent('click');
    });
  });
});
```
