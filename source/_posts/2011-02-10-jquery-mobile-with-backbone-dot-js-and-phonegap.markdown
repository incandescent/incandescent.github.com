---
layout: post
title: "jQuery Mobile with Backbone.js and PhoneGap"
date: 2011-02-10 13:38
comments: true
categories: 
---

We've received a few requests about mobile app development in the past few weeks. We were looking for different options and wanted to use something which could be written once and used on different devices including iOS and Android. We found two solutions: one called <a href="http://www.appcelerator.com/products/titanium-mobile-application-development/">Titanium Mobile</a> and another one called <a href="http://www.phonegap.com">PhoneGap</a>. If you are wondering what the differences between them are, you can read more <a href="http://stackoverflow.com/questions/1482586/comparison-between-corona-phonegap-titanium">here</a>.

We decided to stick with PhoneGap for now mostly because it looked simpler to start with. PhoneGap provides a set of project templates and a common JavaScript API for building "native" mobile applications. It supports iOS, Android, Blackberry, Symbian, and Palm. The JavaScript API allows for interaction with the native API's for things like contact management, camera, GPS and other.

In our previous projects we used a lot of JavaScript. Our code was structured around a great tool called <a href="http://documentcloud.github.com/backbone/">Backbone.js</a>. We also used a lot of jQuery. We wanted to bring this experience to our mobile solution and decided to give jQuery Mobile a try. As a proof of concept we created a simple project called <strong>Happy Pointer</strong>.

<img src="http://incandescent.github.com/happypointer/images/menu.png" />

The app makes use of the data coming from awesome <a href="http://simplegeo.com">SimpleGeo</a> and it's integrated with Google Maps. It basically searches for things near you. (it searches for coffee places by default).

<img src="http://incandescent.github.com/happypointer/images/navigation.png" />

You can find the code for the app on github here: https://github.com/incandescent/happypointer or check the <a href="http://incandescent.github.com/happypointer">demo</a>. jQuery Mobile is very easy to learn. You don't have to write any JavaScript to start with something simple. It comes with numerous useful widgets and components. In order to use them you have to add a "data-role" to your html elements. For example if you want to render a button you can do:

``` html
<a href="index.html" data-role="button">Link button</a>
```

or lets say you want a list:

``` html
<ul data-role="listview"></ul></pre>
```

Overall we think PhoneGap and jQuery Mobile are great solutions for people who already have experience with web technologies and don't want to fight with Android or Cocoa Frameworks. We plan to write more about our experience with Backbone.js and more about our adventures with mobile development in the future so please stay tuned.
