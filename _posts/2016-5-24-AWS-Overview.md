---
layout: post
title: AWS Overview
---

I wanted to learn more about what AWS is and what benefits it can have for a developer. AWS or Amazon Web Services is a cloud platform offering multiple services including computing power, database storage, content delivery. All of these services are available on demand as a service which you pay for as you use them. This makes scaling your application or business a lot easier than a self managed local infrastructure. Let's look at a few great things that AWS offers: 

##### OpsWorks

AWS OpsWorks is a configuration management service that assits in configuring and operating applications using Chef. Chef is an open source infrastructure automation solution, written in Ruby, that allows you to automate the configuration of your systems and the applications that sit on top of it. Below is a simple example of a Chef recipe that installs git:

```ruby
package 'git' do
  action :install
end
``` 

OpsWorks offers templates for common tasks related to application servers and databases but is also fully customizable to build any task that can be scripted. This is definitely a great tool to help automate applications as they change and grow. 

##### Amazon S3

Amazon S3 or Amazon Simplified Storage Service is a highly scalable cloud storage. Amazon S3 is a very easy to use object storage service with an interface that enables you to store and retrieve any amount of data from anywhere on the web. The best part is you only pay for what you use. This is great for scalabilty of an application. Amazon S3 also automatically duplicates your data and stores it on different servers to help protect your data. Amazon S3 also offers different classes of storage depeneding on how frequently the data will be accessed from frequent access to long term archiving, Amazon S3 can do it all. 

##### DynamoDB

Amazon Dynamo DB is a fast and flexible NoSQL database service. NoSQL databases are a high performance and non-relational type of data storage. NoSQL databases utilize a variety of data models, including document, graph, key-value, and columnar. In a relational database you must define the schema before you can add data. This is not the case with a NoSQL database, which allows you to be more agile and scale quickly. It provides consistent single-digit millisecond latency no matter the scale of your application. This cloud based database supports both document and key-value store models. Because of its flexible data model and reliability it is a great fit for almost any type of application. With DynamoDB you do not need to worry about database administration. You can download a local copy onto your machine during development and then use the service to scale globaly. DynamoDB also automatically backs up your data in three seperate facilities. AWS also provides a nice GUI to enable visual creation of tables and items within those tables.

##### Amazon RDS

Amazon RDS or Relational Database Service makes it easy to setup, maintain and scale relational databases in the cloud. It helps manage time consuming database administration tasks so you can focus on development and the business. Amazon RDS includes several different database engines to choose from including : Microsoft SQL Server, Oracle, Amazon Aurora, PostgresSQL, MySQL and MariaDB. There is no need to worry about buying hardware to start or more as you scale. Storage is available whenever you need it in this cloud based service. Amazon RDS allows you to run backups in other regions to help protect your data in case of disaster or downtime. Just like the other AWS services you only pay for what you use. 

##### Amazon CloudFront

Amazon CloudFront is a global content delivery network also known as a CDN. This service can be combined with the other great Amazon Web Services to give developers an easy way to distribute content to end users with low latency, high data transfer speeds, and no minimum use requirements. CloudFront can be used to deliver your entire website including dynamic, static, streaming and interactive content using a global network of edge locations. Content requests are automatically routed to the nearest edge location to allow content to be delivered with the best possible performance. 









