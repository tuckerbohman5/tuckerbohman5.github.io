---
layout: post
title: Adding JQuery To My Rails Application
---


I have been learning so much over the last few weeks. Last weekend I was able to finish building my Task Manager application using ruby on rails. Now I will be adding some additional features using JQuery and converting my rails backend into an API that will communicate with my JQuery front end. This project is another assesment for Learn.co. Here are the requirements for these changes:

1. Must render one show page and one index page via jQuery and an Active Model Serialization JSON Backend.
2. Must use your Rails api to create a resource and render the response without a page refresh.
3. The rails API must reveal at least one has-many relationship in the JSON that is then rendered to the page.
4. Must have at least one link that loads, or updates a resource without reloading the page.
5. Must translate the JSON responses into Javascript Model Objects. The Model Objects must have at least one method on the prototype. Formatters work really well for this. 

I am going to be using the gem `active_model_serializer`. This gives us access to another generator which I will use to create a serializer for projects. `rails g serializer project`. I then will add a few more attributes to the serializer: 

```ruby
class ProjectSerializer < ActiveModel::Serializer
  attributes :id, :name, :description, :due_date
end
```

Then in our projects controller we will add the following to the show action: 

```ruby
respond_to do |f|
      f.html { render :show }
      f.json { render json: @project }
    end
```

Now if we navigate to `/projects/:id.json` we will see the project object in JSON format. This is just the beginning though as we will need to show this information in our views. I decided to add the ability to view the comments on the project show page and then hide them. All this I did without needing to trigure a page refresh. Lets take a look at the javascript required for this task: 

```javascript
$('#loadComments').on('click', function(){
  loadComments();
});


function loadComments() {
    $('#projectComments').html('<h5>Comments:</h5>')
    $.ajax({
      method: 'GET',
      url: this.href,
      dataType: 'JSON'
    }).done(function(response) {
      var project = response["project"];
      var comments = project["comments"];
      for (var i = 0; i < comments.length; i++) {
        var comment = comments[i];
        
        var commentHtml = '';
        commentHtml += "<p>" + comment["user"]["name"] + " said " + comment["content"] + "</p>";
        $('#projectComments').append(commentHtml);
      }
      $('#loadComments').hide();
      $('#hideComments').show();
    })
}
```

When a user clicks on the button with the id of `load comments` it triggers the loadComments function. This uses ajax to send a get request to my rails backend. `this.href` references the current project such as '/project/3' with 3 being the id. When it receives the response in JSON we are able to parse that data by looping through each comment and creating html with that comment and appending it to the project comments div. I then make sure to hide the load comments button and show the hide comments button. I have the hide comments button hidden when the page loads: 

```javascript
$(document).ready(
   function()
   {
     $('#hideComments').hide();
   }
);
```

Another feature I added was the ability to load the description of a project on the home page with a `more info` button. 

```javascript
$('.moreInfo').on('click', function() {
    // event.preventDefualt();
    
    $.ajax({
      method: 'GET',
      url: '/projects/' + this.attributes[1].value,
      dataType: 'JSON'
    }).done(function(response) {
      var id = response["project"]["id"];
      var description = response["project"]["description"];
      $('p[data-id="'+ id +'"]').text(description);
      $('a.moreInfo[data-id="'+ id +'"]').hide();
      $('a.hideInfo[data-id="'+ id +'"]').show();
    })
      })

  $('.hideInfo').on('click', function(){
    var id = this.attributes[1].value;
    $(this).hide();
    $('p[data-id="'+ id +'"]').text('');
    $('a.moreInfo[data-id="'+ id +'"]').show();
  })
```

This uses ajax to load the description of the project and hide it very similar to the load comments functionality. The other feature I added was the ability to view the users and the amount of tasks they have: 

```javascript
$('#viewUsers').on('click', function() {
    $('#viewUsers').hide();
    $('#hideUsers').show();
    $.ajax({
      method: 'GET',
      url: '/users',
      dataType: 'JSON'
    }).done(function(response) {
      var usersDiv = $('#users');
      var users = response["users"];
      
      
      for (var i = 0; i < users.length; i++){
        
        var user = {
          name: users[i]["name"],
          taskCount: users[i]["tasks"].length
        }
          
        usersDiv.append('<p>' + user.name + ' has ' + user.taskCount + ' tasks!</p>');
      }
        
      })

    })
  $('#hideUsers').on('click', function(){
    $('#users').html('');
    $('#viewUsers').show();
    $('#hideUsers').hide();
```

This uses ajax to load all of the users and then loops through each user from the response creating a JS object user for each user in the response. The user object then has two attributes name and taskCount. This information is then displayed appeneded to the users div. Listing username has user.taskCount tasks. I have enjoyed learning Javascript and JQuery and can see great potential in the possible uses I will have for it. I am going to continue to make this application better and definitely add some additionaly CSS to make it more user friendly and looking great. I will also be working on building my first Angular.JS application soon. Until then....






