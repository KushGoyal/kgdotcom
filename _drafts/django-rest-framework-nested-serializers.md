---
layout: post
date: 2020-01-21
title: How to handle nested serializers in Django Rest Framework
categories: [django]
comments: true
---

Django Rest Framework (DRF) does not handle nested json data out of the box. There is an important reason for that.
DRF cannot assume how do you want it to handle the nested data. There are many possible ways to save that data or 
perform business logic over it. There is no defined standard way to do it. So it lets the users decide how to do it. 
But this freedom causes confusion among new developers. 

Stackoverflow has many questions asking the how to save nested 
model data using a model serializer. I don't like most of the answers given there because most of them are very complicated.
Drf documentation has given a small sample code to get flavour of how it should be done. 