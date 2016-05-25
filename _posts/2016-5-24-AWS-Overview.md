---
layout: post
title: AWS Overview
---

I wanted to learn more about what AWS is and what benefits it can have for a developer. AWS or Amazon Web Services is a cloud platform offering multiple services including computing power, database storage, content delivery. All of these services are available on demand as a service which you pay for as you use them. This makes scaling your application or business a lot easier than a self managed local infrastructure. Let's look at a few great things that AWS offers: 

#####OpsWorks

AWS OpsWorks is a configuration management service that assits in configuring and operating applications using Chef. Chef is an open source infrastructure automation solution, written in Ruby, that allows you to automate the configuration of your systems and the applications that sit on top of it. Below is a simple example of a Chef recipe that installs git:

```ruby
package 'git' do
  action :install
end
``` 

OpsWorks offers templates for common tasks related to application servers and databases but is also fully customizable to build any task that can be scripted. This is definitely a great tool to help automate applications as they change and grow. 

#####Amazon S3

Amazon S3 or Amazon Simplified Storage Service is a highly scalable cloud storage. 

