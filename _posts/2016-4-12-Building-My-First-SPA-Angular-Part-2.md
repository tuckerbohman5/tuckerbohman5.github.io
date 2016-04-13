---
layout: post
title: Building My First SPA Using Angular.js(Part 2)
---

In part 2 of this blog post we are going to connect AngularJS to our rails API. Lets get started by first adding the bower-rails gem to our gemfile.

```
gem 'bower-rails'
```

Don't forget to run `bundle install` after updating the gemfile. 

Now we need to create our bower.json file where we will add our dependencies. We can do this by running a rails generator 

```
$ rails g bower_rails:initialize json
```

In our root directory of our application we now have bower.json file. This is where we add our dependencies we need to run our angular application. 

```
{
  “lib”: {
    “name”: “bower-rails generated lib assets”,
    “dependencies”: {
      “angular”: “v1.5.3”,
      “angular-ui-router”: “latest”,
      “angular-resource”: “v1.5.3”
    }
  },
  “vendor”: {
    “name”: “bower-rails generated vendor assets”,
    “dependencies”: {}
  }
}
```

We will be using version 1.5 of angular and the latest version of UI router. After making these changes to the bower.json file we will run `rake bower:install`. This will give us the following folders which contain our angular.js files:

```
lib/assets/bower_components -----> angular
                             |
                             | --> angular-resource
                             |
                             | --> angular-ui-router
```

We now need to require these files in our Javascript Manifest file, but first we need to actually create that file `app/assets/javascripts/application.js` : 

```
//= require angular
//= require angular-ui-router
//= require angular-resource
//= require_tree .
```

We now need to have our template load our Javascript. We can go to app/views/layouts/application.html.erb and add the following line:

```
<%= javascript_include_tag ‘application’ %> 
```

Alright our next step is to actually make our application do something so lets go ahead and create a new home page for our web app. In our config/routes.rb lets add a new root route. 

```
root "application#index"
```

This will trigger the index method in the application controller which means we now need to add that method to our application_controller.rb. 

```
def index
end
```

Then we need to create the file app/views/application/index.html.erb which can be blank for now. Now if we start our rails server we will see a blank webpage in the browser instead of the default rails landing page. We are now ready to start implementing AngularJS. Lets start by editing the app/views/layouts/application.html.erb and add angular to our base template:

```
<body ng-app="app">
```

This will initialize our angular module 'app' into our application, but wait a minute we haven't created any angular modules. Lets do that now. 






