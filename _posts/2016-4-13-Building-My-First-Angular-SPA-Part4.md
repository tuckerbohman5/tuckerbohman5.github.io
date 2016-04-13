---
layout: post
title: Building My First SPA Using Angular.js(Part 4)
---

In this part of the blog post we are going to use ngResource to make calls to our rails API. (Make sure you have completed the first three parts or this is not going to work.) Lets get started by first wiring up our state('books') to our database so that we can see all the books we have stored in our database. To do this we will need to create a factory in the following new file app/assets/javascripts/angular-app/models/book.js: 

```javascript
angular 
  .module('app')
  .factory('Book', Book)
function Book($resource) {
  
  var Book = $resource('http://localhost:3000/api/v1/books/:id.json', {id: '@id'}, {
  update: { method: 'PUT' }
});
  return Book; 
}
```

Notice that we will be using $resource inside of our model. Now that we have a book factory we can use the query() method to get all of the books in our database. Lets make those changes in our BooksController.js: 

```javascript
angular
  .module('app')
  .controller('BooksController', BooksController);
function BooksController() {
  var ctrl = this;
  ctrl.books = Book.query();
}
```

Here we changed the ctrl.books from an array to the result of querying our books table. 