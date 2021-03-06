---
layout: post
title: Building My First SPA Using Angular.js(Part 1)
---


This will be the first in a series of blog posts as I build my final project for Learn.co. I am going to SPA(single page application) that allows teachers to keep track of the many books they have in their classroom library. This will be similar to the Sinatra Library I built earlier this year except it will be more adapted for the classroom use by teachers. We will be communicating to a Rails backend using a JSON API. To get this started I will be following along with a fellow classmates blog which is really great and explains in detail how to do this. I encourage you to check it out if you get stuck like I did. [4-Part Series on Creating an Angular.JS App with Rails restfulAPI](https://medium.com/@lukeghenco) 

I am going to start with building the Rails backend and then we will move into building an awesome Angular.JS front-end down the road. First things first lets create our basic rails app using the rails new generator: 
`$ rails new library --database=postgresql --skip-javascript` 
Note: We are using a postgres database and we will skip adding javascript for now since we will be adding Angular.JS to our application later. Now that we have the basics its time to start building the models for our library. Lets take a few minutes and decide exactly what models and what properties those models should have in our library. 

![Picture of database plan](http://tuckerbohman5.github.io/images/ng-lib-db.jpg "Database Plan For Library")

This is just the two main tables teachers who will be the users of this application and the books which is what this library is all about. 

Lets create these two models first and then we can add all of the smaller tables. Lets start with the teacher model: 
`rails g model Teacher title:string first_name:string last_name:string grade_id:integer school_id:integer email:string password_digest:string`
Then the book model:
`rails g model Book title:string author_id:integer reading_level_id:integer teacher_id:integer`
We will add category later on when we decide exactly how the relationship should be. For now lets go ahead and generate the other simple tables we will need. School, Author, Reading Level, and Grade. Then we will migrate our database and create some seed data. Creating seed data is tedious and takes time but it will be very helpful to test the functionality of my application. 

Now we will build our RestfulAPI starting in the config/routes.rb file:

```ruby
Rails.application.routes.draw do
  namespace :api, defaults:{format: :json} do
    namespace :v1 do
      resources :teachers, :books, :authors, :grades, :reading_levels, :schools
    end
  end
end
```

This will create our API as well as versioning for our API. We are chosing to format the data to json by default to make it easy to interact with that data in our Angular front end. If you look at the routes by running `rake routes` you will also see that we have a basic CRUD application available to us now. Now that we have our routes we need to create controllers for each of our models. Let the fun begin. It is important to remember that because we have our routes nested inside the api/v1 namespace we will need to create our controllers in within controllers/api/v1. Some of our models will be very complicated when it comes to different controller actions but lets go ahead and look at the basics. controllers/api/v1/schools_controller.rb:

```ruby
module Api
  module V1 
    class SchoolsController < ApplicationController 
      skip_before_filter :verify_authenticity_token 
      respond_to :json 
      def index 
        respond_with(School.all.order("id DESC"))
      end 
      def show 
        respond_with(School.find(params[:id]))
      end 
      def create 
        @school = School.new(school_params) 
        if @school.save 
          respond_to do |format|
            format.json { render :json => @school }
          end 
        end 
      end 
      def update 
        @school = School.find(params[:id])
        if @school.update(school_params) 
          respond_to do |format| 
            format.json { render :json => @school }
          end 
        end 
      end 
 
      def destroy 
        respond_with School.destroy(params[:id]) 
      end 
      private 
        def school_params 
          params.require(:school).permit(:name) 
        end 
    end 
  end
end
```

Now if we start the rails server `rails s` and navigate to localhost:3000/api/v1/schools we will see the JSON for all of the schools. Pretty awesome right? Well we are just getting started. I will go ahead and create the rest of the controllers we need but we now have a basic API set up. Now what? In the next post we will look into how exactly we connect our rails API to angular.JS. 


