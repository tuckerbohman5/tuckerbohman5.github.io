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

When we use Angular with rails we are going to place all of our views inside of a `templates` folder. However, how can we tell rails about this templates folder? Lucky for us, someone already went through the hard work and create a gem for us to use so all we need to do is add the following to our gemfile and run bundle install. 

```
gem 'angular-rails-templates'
```

Then we will add the following to our javascript manifest(app/assets/javascripts/application.js): 

```javascript
//= require angular-rails-templates
```

The last step is to inject 'templates' into our 'app' module in our app.js file: 

```javascript
angular
 .module('app', ['ui.router', 'ngResource', 'templates'])
```

Lets create the following files that we specified in app.js as our template html files: 

```
app/assets/javascripts/templates/home.html
app/assets/javascripts/templates/home/new.html
app/assets/javascripts/templates/home/books.html
```

Now lets add some html to our templates so they aren't so boring. I am going to start with home.html:

```html 
<div class="navbar">
  <h1>Angular Classroom Library</h1>
  <hr>
  <br>
  <a href="#" ui-sref="home.new" ui-sref-active="active">New Note</a>
  <a href="#" ui-sref="home.books" ui-sref-active="active">My Books</a>
</div>
<br>
<hr>
<div ui-view></div>
```

We have two links with the ui-sref linking to the other two states. The ui-sref-active attribute changes the class of the link to active when it is the current state. The bottom div ui-view is where the nested states will load their html when state changes. We can add some css to see these changes in our app/assets/stylesheets/application.css: 

```css
/*
 *= require_tree .
 *= require_self
 */
a {
  text-decoration: none;
  color: white;
  padding: 8px;
  background-color: green;
  border: solid 2px black;
}
.active {
  background-color: blue;
}
```

This will make the background color blue when the links state is active and green when it is not. Now we can move on to updating our other two templates. Remember that we are making everything very simple right now and then we will make everything more complex, but for now we just want to get angular and rails working. In our app/assets/javascripts/templates/home/new.html lets add the following: 

```html
<h1>New book</h1>
<form>
 <label for="title">Title</label>
 <input name="title">
 <label for="author">Author</label>
 <input name="author">
 <input type="submit" value="Add Book">
</form>
```

In our app/assets/javascripts/templates/home/books.html lets add a simple view for our books.

```html
<h1>Notes</h1>
<ul>
  <li ng-repeat="book in ctrl.books">
    <h3>{{ book.title }}</h3>
    <p>
      {{ book.author }}
    </p>
  </li>
</ul>
```



