---
layout: post
title: Building My First SPA Using Angular.js(Part 3)
---

Things are going to start getting more complicated so take your time and go slow through these next few steps. We are going to start by creating routes using ui.router. Lets first remove the default text we created in app/views/application/index.html.erb and replace it with: 

```
<div class="angular-view-container" ui-view></div>
```

This ui-view directive tells ui.router where it should load the templates it receives for each state. Lets now open up our app/assets/javascripts/angular-app/app.js and add the following changes: 

```javascript
angular
 .module('app', ['ui.router', 'ngResource'])
 .config(function($stateProvider, $urlRouterProvider) {
   $stateProvider
     .state('home', {
       url: '/',
       templateUrl: 'home.html',
       controller: 'HomeController as ctrl'
     })
     .state('home.new', {
       url: 'new',
       templateUrl: 'home/new.html',
       controller: 'BooksController as ctrl'
     })
     .state('home.books', {
       url: 'books',
       templateUrl: 'home/books.html',
       controller: 'BooksController as ctrl'
     });
  $urlRouterProvider.otherwise('/');
});
```

We now have created three basic routes using $stateProvider. $stateProvider is a service built into Angular that allows us to call routes using the .state function. It takes in two arguments: 1) Our route name 2) An object with our keys each state will use such as the url, templateUrl, and controller. We will need to create our templateUrls and controllers but first it is important to note that the home.new and home.books routes are nested routes so in our browser they will show localhost:3000/#/new and localhost:3000/#/books. We will get into building our controllers in just a bit but first we need to fix an issue with rails. 

