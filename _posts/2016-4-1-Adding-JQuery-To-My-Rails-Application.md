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

I am going to be using the gem `active_model_serializer`. This gives us access to another generator which I will use to create a serializer for projects. `rails g serializer project`. 