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
Drf documentation has given a small sample code to get a flavour of how it should be done.

The most common example of nested serializer is inner model having the foreign key relation to the outer model. For example
an invoice with many invoice lines.

Creation is straight forward; you have to create all the invoice lines when saving the invoice. Updating has several different 
cases:
- Create new invoice lines which did not exist before (id == None)
- Update invoice lines which still exist (id != None)
- Delete invoice lines which are no longer in the list

To handle the above cases you have to modify both the create and update method on the serializer of the parent.

First lets look at the parent serializer code.

```python

```
