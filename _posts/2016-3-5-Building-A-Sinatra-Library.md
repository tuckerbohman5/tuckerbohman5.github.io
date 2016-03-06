---
layout: post
title: Building A Sinatra Library
---


I have been learning a lot over the last few months about ORMs, Active Record, Rack, and Sinatra. I am now ready to build my very own Sinatra project. This will be my second project in the Flatiron Learn cirriculum. The requirements for this project are as follows: 

1. Build an MVC Sinatra Application.
2. Use ActiveRecord with Sinatra.
3. Use Multiple Models.
4. Use at least one has_many relationship
5. Must have user accounts. The user that created the content should be the only person who can modify that content
6. Models must have validations to ensure that bad data isn't created
7. Any validation failures must be shown to user with an error message

The first step is I must decide what I want to build and start outlining the requirements for my idea: 
I know I want to build a home library to keep track of all of the books in my house. My wife and I both love books so we have a lot. I think it would be fun to build a little application that allows you to add books, authors and keep track of all the books I have.
To start I know that I will want to have the following models:
* User - which will require a username, password and allow them to login and see the books they own. 
* Book - Each book will have a title, and belong to an author. 
* Author - An author will have a name and have many books. 

Lets start building out the basic Sinatra application structure and build the different models. 
I simply used a previous project to see which files were needed in my new application repository I created. Once I had the basic file structure in place it was time to begin coding. Lets start building the model structure for Books, Authors, and Users. 

Book
1. `rake db:create_migration NAME="create_books"`
2. Modify the migration file. 

```ruby 
class CreateBooks < ActiveRecord::Migration
  def change
    create_table :books do |t|
      t.string :title
      t.integer :author_id
      t.integer :user_id
    end
  end
end
```

3. Create a Book class for the model. 

```ruby
class Book < ActiveRecord::Base
  belongs_to :author
  belongs_to :user
end
```

I then repeated this process to create the Author and User models. 
Don't forget to run `rake db:migrate` to actually build your tables. 

Now lets create a home page that welcomes and allows the user to sign in or sign up. 
We will start adding this functionality in the application controller. 
`contollers/application_controller.rb` 

```ruby 
require '.config/environment'

class ApplicationController < Sinatra::Base
  configure do 
    enable :sessions
    set :session_secret, 'secret_family_library'
  end

  get '/' do 
    erb :home
  end
end
```

Lets break this down a little we are first requiring our environment. 
The next block of code is really important because it allows our application to access the sessions hash using the session keyword. Without this functionality it would be impossible to keep track of the current user and they would have to login everytime they went to a different part of the application. When the user logs in their id is stored in the sessions hash with the :id key. 

```ruby
  configure do 
    enable :sessions
    set :session_secret, 'secret_family_library'
  end
```
The next part of our code simply routes all requests to the main page of our application to the homepage which will be rendered in erb by the browser. Lets add a simple welcome and the functionality to login or sign up to our homepage. I added the simple functionality like so using a form to login or a link to the sign up page. 

```html
<h1>Welcome to the Sinatra Home Library!</h1>
<p>Add description here...</p><br>

<form action="/login" method="POST">
  <p>Username: <input type="text" name="username" required="required"></p>
  <p>Password: <input type="password" name="password" required="required"></p>
  <p><input type="submit" value="Log In"></p>
</form>

<p>If you don't already have an account <a href="/signup">Sign Up Here</a></p>
```

I created a new controller to keep specifically for actions having to do with the user. 

```ruby
class UsersController < ApplicationController
  post '/login' do 
  end

  get '/signup' do   
  end
end
```

It will inherit from ApplicationController which inherits from Sinatra::Base. It is also important to add the following line to your config.ru file for each controller. `use UsersController` This will allow your application to use the other controllers and not just the ApplicationController. 

I added the following route to the UsersController

```ruby
post '/login' do 
    @user = User.find_by(username: params[:username])

    if @user && @user.authenticate(params[:password])
      session[:user_id] = @user.id
      redirect '/books'
    else
      erb :'/login'
    end
  end
```

When the user submits the form to login it will look in the database of users to find a match by username. The if statement looks for two things to be true: 1. The database search was able to find a match and save it in the @user instance variable and 2. it checks to see if the password the user entered matches the correct password for the user. If it does it will take them to the home page. If not it will take them back to the login page. 


Lets build the Sign Up Page: 

```html
<h1>Please enter your information to create an account with Sinatra Home Library</h1><br>

<form action="/signup" method="POST">
  <p>Username: <input type="text" name="username" required="required"></p>
  <p>E-Mail: <input type="text" name="email" required="required"></p>
  <p>Password: <input type="password" name="password" required="required"></p>
  <input type="submit" value="Sign Up">
</form>
```

Then we need to add a POST route to our controller:

```ruby
post '/signup' do 
    #I will need to add additional validation later :)
    @user = User.create(params)
    session[:user_id] = @user.id
    redirect '/books'
  end
```

Now users are able to login or signup. Now on to the next problem of trying to allow users to create books and authors. This will be fun. 

Alright so lets look at the form needed to create a new book: 

```html
<h1>Add a new book to your library</h1>
<form action="/books" method="POST">
  <p>Title</p>
  <input type="text" name="title"><br>
  <p>Author</p>
  <p>Select an existing author: </p>
  <select name="author_id">
    <% @authors.each do |author| %>
      <option value=<%=author.id %>><%= author.display_name %></option>
    <% end %>
  </select>
  <p>Or create a new one: </p>
  <p>Author First Name: <input type="text" name="author[first_name]">
  <p>Author Last Name: <input type="text" name="author[last_name]"></p>
  <input type="submit" value="Add Book">
</form>
```

So in this we give the user the ability to add books to their library. It has the ability to select an existing autor in the drop down or create a new author I just got done testing this and it works after a lot of tweaking. Lets take a look at the controller for books to see how I implemented this functionality:

```ruby 
class BooksController < ApplicationController
  get '/books' do
    erb :'/books/index'
  end
  
  get '/books/new' do 
    @authors = Author.all
    erb :'/books/new'
  end

  post '/books' do 
    @book = Book.new
    @author = Author.find(params[:author_id])
    if @author 
      @book.author = @author
    else 
      @author_new = Author.create(params[:author])
      @book.author = @author_new
    end
    @book.title = params[:title] 
    @book.save
    redirect "/books/#{@book.id}"
  end

  get '/books/:id' do 
    @book = Book.find(params[:id])
    erb :'/books/show'
  end
end
```

As you can see when the user submits the form for a new book it creates a new instance of the book object and then tries to find the author of the book from the drop down or if it can't find the author it will create a new author. It then sets the title of the book and saves the book to the database. It will then take the user to the show page for that book(which is still a work in progress). 

After some additional testing I found a bug in my code. The current form setup allows you to select an existing author but because the drop down was always on an author it would choose that author always which prevented me from creating a new author. So I made some changes to the form and the controller. 

The form: 

```html 
<p>Select an existing author: </p>
  <select name="author_id">
      <option value="999999">Create new...</option>
    <% @authors.each do |author| %>
      <option value=<%=author.id %>><%= author.display_name %></option>
    <% end %>
  </select>
```

I added another option to the selector for Create new... And then I changed the logic of the controller: 

```ruby
post '/books' do 
    @book = Book.new
    if params[:author][:first_name] != ""
      @book.author = Author.create(params[:author])
    else
      @book.author = Author.find(params[:author_id])
    end
     @book.title = params[:title] 
    @book.save
    current_user.books << @book
    redirect "/books/#{@book.id}"
  end
  ```

This checks to see if the fields are empty first and if empty it finds the existing author, if it has a value it will create the new author. 

I updated the index page:

```html
<h1><%=current_user.username%>'s Library</h1>
<% current_user.books.each do |book| %>
  <p><a href="/books/<%=book.id%>"><%= book.title %></a><p><br>
<% end %>
<form action="/logout" method="get">
<button>Log Out</button>
</form>
```
And updated the controller to allow logout functionality: 

```ruby
get '/logout' do 
    session.clear
    redirect '/'
end
```

I added the ability to edit books with a form and a controller action. 

```html
<h1>Edit Book</h1>
<form action="/books/<%=@book.id%>" method="POST">
  <p>Title</p>
  <input type="text" name="title" value="<%= @book.title%>"><br>
  <p>Author</p>
  <p>Author First Name: <input type="text" name="author[first_name]" value="<%= @book.author.first_name%>"></p>
  <p>Author Last Name: <input type="text" name="author[last_name]" value="<%= @book.author.last_name%>"></p>
  <input type="submit" value="Edit Book">
</form>
```

```ruby
get '/books/:id/edit' do 
    @book = Book.find(params[:id])
    erb :'/books/edit'
  end

  post '/books/:id' do 
    @book = Book.find(params[:id])
    @book.update(title: params[:title])
    @book.author.update(params[:author])
    redirect "/books/#{@book.id}"
  end
```

I added delete functionality to the controller: 

```ruby
get '/books/:id/delete' do 
    @book = Book.find(params[:id])
    if current_user.id == @book.user_id
      @book.delete
      redirect '/books'
    else
      #error message
      redirect '/books'
    end
```

I added some validation features to the new book action: 

```ruby
post '/books' do 
    redirect_if_not_logged_in
    @book = Book.new
    if params[:author][:first_name] != "" && params[:author][:last_name] != ""
      @book.author = Author.create(params[:author])
    elsif params[:author_id] == "999" 
            redirect "/books/new?error=Please select or create a valid author"
    elsif @author = Author.find(params[:author_id])
      @book.author = @author
    else
      redirect "/books/new?error=Please select or enter a valid author"
    end
      
     @book.title = params[:title] 
     if @book.title = ""
      redirect "/books/new?error=Please enter a title"
      else
     @book.save
   end
```

Then I added additional validation for the edit book action: 

```ruby
get '/books/:id/edit' do 
    redirect_if_not_logged_in
    @book = Book.find(params[:id])
    @error_message = params[:error]
    erb :'/books/edit'
  end

  post '/books/:id' do 
    redirect_if_not_logged_in
    @book = Book.find(params[:id])
    if params[:title] != ""
    @book.update(title: params[:title])
    else
      redirect "/books/#{@book.id}/edit?error=Please enter a valid title!"
    end
    if params[:author][:first_name] != "" && params[:author][:last_name] != ""
    @book.author.update(params[:author])
    redirect "/books/#{@book.id}"
    else
      redirect "/books/#{@book.id}/edit?error=Please enter a valid author!"
    end
  end
```






