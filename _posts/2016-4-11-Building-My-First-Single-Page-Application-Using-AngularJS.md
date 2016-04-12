---
layout: post
title: Building My First SPA Using Angular.js
---


This will be the first in a series of blog posts as I build my final project for Learn.co. I am going to SPA(single page application) that allows teachers to keep track of the many books they have in their classroom library. This will be similar to the Sinatra Library I built earlier this year except it will be more adapted for the classroom use by teachers. We will be communicating to a Rails backend using a JSON API. To get this started I will be following along with a fellow classmates blog which is really great and explains in detail how to do this. I encourage you to check it out if you get stuck like I did. [4-Part Series on Creating an Angular.JS App with Rails restfulAPI](https://medium.com/@lukeghenco) 

I am going to start with building the Rails backend and then we will move into building an awesome Angular.JS front-end down the road. First things first lets create our basic rails app using the rails new generator: 
`$ rails new library --database=postgresql --skip-javascript` 
Note: We are using a postgres database and we will skip adding javascript for now since we will be adding Angular.JS to our application later. Now that we have the basics its time to start building the models for our library. Lets take a few minutes and decide exactly what models and what properties those models should have in our library. 

![Picture of database plan](./images/ng-lib-db.jpg "Database Plan For Library")
