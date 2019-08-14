---
layout: post
title: "WEBCAMPS Sydney Day 1 / Part 1"
date: 2010-06-07 01:39
author: ahmadreza
comments: true
categories: [Blog]
tags: [Australia, Sydney, WEBCAMPS]
---


WEBCAMP was a 2 days event that provides a chance to learn and build projects based on Microsoft Web Platform. WEBCAMPS is held in different cities around the word and I had chance to participate in Sydney WEBCAMP event in 28 and 29 of May 2010. Interesting point is performers of this event in Sydney were [Scott Hanselman](https://hanselman.com/) and [James Senior](https://www.jamessenior.com/) who I really wanted
  

to be in one of their presentations specially Scott that you’ll never be tired in a conference in case Scott is one of presenters.
[https://www.asp.net/dynamicdata](https://www.asp.net/dynamicdata)
  

These project types are useful for creating administration pages from scratch. All you need to do is create your database and relations. Create a project type either Entity of Linq to SQL and select entities you want. Suddenly you find that All CRUD (Create/Remove/Update/Delete) of your entities is created in user friendly URL by using “DynamicDataRoute” class.
  

Following is steps to create an administration web site for Northwind in less than 2 minutes:
  

1. Create a new web project type Asp.net Dynamic Data Entities Web application.
2. Copy Nothwind Database files (MDF and LDF) in APP_DATA folder.
3. Right click on project and new data item type ADO.Net Entity Data Model.
4. In wizard form select “Generate from database” item.
5. And then select NORTWIND as data connection.
6. Select tables you want to be added to Entity Data Mod and then click finish.
7. Open Global.asax and then remark line starting with DefaultModel change this line as follow.


```C#
DefaultModel.RegisterContext(typeof(NorthwindEntities), new ContextConfiguration() { ScaffoldAllTables = true });
```


8.Press F5 and run the application.




Simple, Easy and beautiful, It’s a good solution for quick data administration generation. And the good thing is all templates are editable and you can change them.

