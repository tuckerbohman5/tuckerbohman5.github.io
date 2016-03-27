---
layout: post
title: My First Rails Application
---

Today I will be starting to build my first Rails application completely from scratch. I have been excited to start this project because just like the home library I will be building something I can actually use. For this project I am going to build a team task/project manager that my team at my current job can keep track of to use of current projects and make sure nothing slips past the cracks. This project will also fulfill the requirements for my third assesment for Learn.co. The requirements for this project are the following: 

1. Use the Ruby on Rails framework.
2. Your models must include a has_many, a belongs_to, and a has_many :through relationship. You can include more models to fill out your domain, but there must be at least a model acting as a join table for the has_many through.
3. The join model must also store an additional attribute describing the relationship. For example, in a blog domain with comments by users, you'd have a posts table and a users table, with the comments table containing the foreign key for the post_id and the user_id along with the comment's content. In a TODO list application with shareable lists, you'd have a lists table and users table and then a user_lists table giving users access to lists via columns user_id and list_id, but you'd want to add a permission column to user_lists that described how a user relates to the list, whether they could edit it or just view it or delete it, etc.
4. Your models should include reasonable validations for the simple attributes. You don't need to add every possible validation or duplicates, such as presence and a minimum length, but the models should defend against invalid data.
5. You must include at least one class level ActiveRecord scope methods. To some extent these class scopes can be added to power a specific individual feature, such as "My Overdue Tasks" in a TODO application, scoping all tasks for the user by a datetime scope for overdue items, @user.tasks.overdue. Reports make for a good usage of class scopes, such as "Most Valuable Cart by Customer" where the code would implement a Cart.most_valuable and Cart.by_customer which could be combined as Cart.most_valuable.by_customer(@customer).
6. You must include a nested form that writes to an associated model through a custom attribute writer. An example of this would be a New Recipe form that allowed you to add ingredients that are unique across recipes (thereby requiring a join model, or imagine being able to see all recipes that include Chicken), along with a quantity or description of the ingredient in the recipe. On this form you would have a series of fields named recipe[ingredient_attributes][0][name] and recipe[ingredient_attributes][0][description] which would write to the recipe model through a method ingrediebt_attributes=. This method cannot be provided via the accepts_nested_attributes_for macro because the custom writer would be responsible for finding or creating a recipe by name and then creating the row in the join model recipe_ingredients with the recipe_id, the ingredient_id, and the description from the form.
7. Your application must provide a standard user authentication, including signup, login, logout, and passwords. You can use Devise but given the complexity of that system, you should also feel free to roll your own authentication logic.
8. Your authentication system should allow login from some other service. Facebook, twitter, foursquare, github, etc...
9. You must make use of a nested resource with the appropriate RESTful URLs. Additionally, your nested resource must provide a form that relates to the parent resource. Imagine an application with user profiles. You might represent a person's profile via the RESTful URL of /profiles/1, where 1 is the primary key of the profile. If the person wanted to add pictures to their profile, you could represent that as a nested resource of /profiles/1/pictures, listing all pictures belonging to profile 1. The route /profiles/1/pictures/new would allow to me upload a new picture to profile 1.
10. Your forms should correctly display validation errors. Your fields should be enclosed within a fields_with_errors class and error messages describing the validation failures must be present within the view.
11. Your application must be, within reason, a DRY (Do-Not-Repeat-Yourself) rails app. Logic present in your controllers should be encapsulated as methods in your models. Your views should use helper methods and partials to be as logic-less as possible. Follow patterns in the Rails Style Guide and the Ruby Style Guide. 

Now the hardest part of all this which is going to be determining which models will be needed and their associations. I know the following is true: 

* This application will require users, which will have a `name`, `email`, and `password.` 
* This application will have Projects with a `title`, `due_date`, has_many `tasks`, has_one `project_manger`, has_many `comments`.
* Tasks can belong to a project or they can be seperate, they belong_to a user. 
* Comments will belong to a project and a task. 

I am sure the exact structure will probably change but for now lets start building one step at a time. Lets build the `User` model. 
I will start by building the login/signup/logout functionality without using Devise or any other similar gem. Let the fun begin: 

`rails g model User name:string email:string password_digest:string` 

Rails has a great feature built in called a generator that can be used to build all sorts of great things including models, migrations, resources, and even the magical scaffold(which is so magical you should probably never use it.) That being said in the above line that we ran in the terminal it will create a User model for us and a migration file for the database. Lets take a look: 

```ruby 
class CreateUsers < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.string :name
      t.string :email
      t.string :password_digest

      t.timestamps null: false
    end
  end
end
```

As you can see the model created the migration with the attributes we specified. The power of rails is real! Now we can go ahead and run `rake db:migrate` to update the schema of our database. Lets see what the User class looks like:

```ruby
class User < ActiveRecord::Base
  has_secure_password
end
```

I added the has_secure_password for the authentication functionality built into rails. Now lets start creating some views and the functionality needed for the user to log in and out or signup. We will then later add the `omniauth` functionallity to allow users to login via facebook. 


I am not going to go into all all the details of how I created my entire application but you can see the source code here: [Fuego Task Manager](https://github.com/tuckerbohman5/rails-team-task-manager). After a week of coding the Application is finally ready to submit. However before I end this blog post I just wanted to mention one thing I learned during the struggle. I was really struggling to get Omniauth(login with Facebook) functionailty to work with my custom authentication. Here is what I was finally able to get to work. 

```ruby
def create
    
    if auth.nil?
      @user = User.find_by(email: params[:user][:email])
      if @user && @user.authenticate(params[:user][:password])
      session[:user_id] = @user.id 
      redirect_to root_path
      else
      @error = "Invalid Login Please Try Again"
      render :new
      end
   
    else 
      @user = User.find_or_create_by(uid: auth['uid']) do |u|
        u.name = auth['info']['name']
        u.email = auth['info']['email']
      end
      
      @user.save
      session[:user_id] = @user.id
      redirect_to root_path

    end
  end 
```

My create method first checks to see if the return of auth is nil. auth is a method of omniauth that returns the credentials of a user after logging in to Facebook. If the user did not use Facebook it will validate that they are a valid user in the database and authenticate using the password they entered vs the password stored securily in the database. If the information matches they will be logged in storing the User's id in the session. If not they will be redirected to the login path with an error message. If they did use Facebook to login it will find or create a new user based on the uid sent to us from Facebook. That User's id is then stored in the session. The tricky part of all of this is that when logging in with Facebook the user does not have a password. In my user model I had 'has_secure_password' so active record was preventing the record from saving. I finally came up with a fix for this with the following code in the User model. 

```ruby
  before_save :social_login?
  
  def social_login?
    if self.uid.nil?
      has_secure_password
    end
  end
```

So the active record callback before_save calls the method :social_login? before it tries to save the record. If the User has an uid attribute it will not set require a password. This allows my application to allow for both normal login with email and passwords as well as logging in with facebook. 

Overall this project has been a lot of fun to build and I have learned so much. This week I will be learning about adding Javascript and JQuery to my rails application and I will write another blog post about that process as well. 

I celebrated early but I finally found a fix for user that works: 

```ruby
  has_secure_password 
  before_save :social_login?

  def social_login?
    if uid
      password = "f@c3b00k"
    end
  end
```

I think the long term solution for best security is going to be implementing the Devise gem but for now this is going to have to do.









