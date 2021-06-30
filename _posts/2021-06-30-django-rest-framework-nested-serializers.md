---
layout: post
date: 2021-06-30
title: Nested Serializers in Django Rest Framework
categories: [django]
image: /images/2021-06-30/nest.jpg
---
[Django Rest Framework](https://www.django-rest-framework.org) (DRF) does not handle nested json data out of the box. There is an important reason for that. DRF cannot assume how do you want it to handle the nested data. There are many possible ways to save that data or perform business logic over it. There is no defined standard way to do it. So it lets the users decide how to do it. But this freedom causes confusion among new developers. 

StackOverflow has many questions asking the how to save nested model data using a model serializer. I don’t like most of the answers given there because most of them are very complicated. DRF documentation has already given a small [sample code](https://www.django-rest-framework.org/api-guide/relations/#writable-nested-serializers) to give a flavour of how it should be done.
<!--more-->

The most common example of nested serializer is inner model having the foreign key relation to the outer model. For example a sales invoice with many line items. Line item model has a foreign key field to the invoice model.

Creation is straight forward; you have to create all the line items when saving the invoice. But updating has many cases:
- Create new line item which did not exist before `line_item.id == None`
- Update line items which already exist `line_item.id != None`
- Delete line items which are no longer present in the POST data

To handle the above cases you have to modify both the create and update method on the serializer of the parent.

The parent serializer is the Invoice serializer and the child serializer is the line item serializer. Parent serializer will inherit the nested serializer behaviour. Below is the code for the nested serialiser.

<script src="https://gist.github.com/KushGoyal/5b72c5986fd6713fbf074dabca337a9d.js?file=nested_serializer.py"></script>

Create and update methods are very similar. In both we are removing the nested child data from the `validated_data` dict,  then using the **list child serializers** to create or update the list of child data. This is done for every child field in a for loop.

`atomic` decorator is used to revert any saves in the database in case there is any validation error in the child data.

Now let’s see the child list serializer code. We have created a list serializer which contains all the logic to update and create the child objects using the list data.

Create method is not modified since create is straight forward. Update method is modified the handle the creation, updates and deletion of child objects within the update request of the parent model.

<script src="https://gist.github.com/KushGoyal/5b72c5986fd6713fbf074dabca337a9d.js?file=list_serializer.py"></script>

In the above code the `set_parent_data` is used to set data from the parent object on the child object by adding it the data dicts. This is done so that the API consumer does not have to supply the same data again and again in the list of child data. `set_parent_data` is called in the parent serialiser create and update method to set the values in the child data before saving the child data.

`from_parent_fields` is set on the meta class of the child serializer class. This is a list of fields which are copied from the parent instance to the child instances.

`foreign_key_field_name` is set on the meta class of the child serializer as the name of the foreign key field of the parent model on the child model. For example the child model `line_item` has foreign key field `invoice` on it, so the value of `foreign_key_field_name` will be `invoice`.

To the use the above list serializer class you have to set it on the child serialiser class as the `list_serializer_class` in the meta class of the child serializer.

Below is an example of usage of the above serializer classes. The parent model is `PurchaseOrder` and the child model is the purchase order line item called  `Poline`.

<script src="https://gist.github.com/KushGoyal/5b72c5986fd6713fbf074dabca337a9d.js?file=example.py"></script>