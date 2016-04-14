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
function BooksController(Book) {
  var ctrl = this;
  ctrl.books = Book.query();
}
```

Here we changed the ctrl.books from an array to the result of querying our books table. It is also important to make sure we pass in Book to BooksController. We will now get a list of all the books in our database in the my books page. But it isn't exactly working because we do not have the author data coming across in our json. Lets work on fixing this next. Alright I figured out a fix for this issue. In my book.rb file I added the following: 

```ruby
def as_json(options = {})
    super(options.merge(include: [:author, :reading_level, :teacher]))
end
```

We are overwritting the default as_json method with our own custom method. This allows us to include associations of the book model such as author, reading_level, and teacher. Now if we edit our books.html with the following: 

```html
<h1>Books</h1>
<ul>
  <li ng-repeat="book in ctrl.books">
    <h3>{{ book.title }}</h3>
    <p>
      {{ book.author.last_name + ', ' + book.author.first_name }}
    </p>
  </li>
</ul>
```

Notice we are calling the author associated with the book's name and adding a little formatting so that it is lastname, firstname. This will now work. 

