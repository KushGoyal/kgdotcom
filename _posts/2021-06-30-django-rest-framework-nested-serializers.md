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

```
from django.contrib.contenttypes.models import ContentType
from django.db.transaction import atomic
from rest_framework import serializers


class NestedSerializerMixin(serializers.Serializer):

    @atomic
    def create(self, validated_data):
        list_data = {}
        # remove the child data from the validated_data
        for key, val in self.get_fields().items():
            # find the child fields and remove the data from validated_data dict
            if isinstance(val, serializers.ListSerializer) and not val.read_only:
                list_data[key] = validated_data.pop(key, [])
        instance = super().create(validated_data)
        # for each child data initialize the child list serializer and create the child objects
        for key, data in list_data.items():
            # noinspection PyTypeChecker
            serializer: ListUpdateSerializer = self.get_fields()[key]
            serializer.set_parent_data(data, instance)
            serializer.create(data)
        return instance

    @atomic
    def update(self, instance, validated_data):
        list_data = {}
        for key, val in self.get_fields().items():
            if isinstance(val, serializers.ListSerializer) and not val.read_only:
                list_data[key] = validated_data.pop(key, [])
        instance = super().update(instance, validated_data)
        for key, data in list_data.items():
            # noinspection PyTypeChecker
            serializer: ListUpdateSerializer = self.get_fields()[key]
            serializer.set_parent_data(data, instance)
            serializer.update(getattr(instance, key).all(), data)
        return instance
```

Create and update methods are very similar. In both we are removing the nested child data from the `validated_data` dict,  then using the **list child serializers** to create or update the list of child data. This is done for every child field in a for loop.

`atomic` decorator is used to revert any saves in the database in case there is any validation error in the child data.

Now let’s see the child list serializer code. We have created a list serializer which contains all the logic to update and create the child objects using the list data.

Create method is not modified since create is straight forward. Update method is modified the handle the creation, updates and deletion of child objects within the update request of the parent model.

```
class ListUpdateSerializer(serializers.ListSerializer):

    def set_parent_data(self, data, parent, parent_field=None):
        """
            This method is used to set data on the child objects from the parent object
            
            There are 3. fields you can set on the meta class of the child serializer class:
            1. from_parent_fields: this field is the list of common fields between the parent
                                   and child model. The values are copied from parent to child
            2. generic_foreign_key: this field is tuple of object_id and content_type_id field
            3. foreign_key_field_name: this field is the field name of the foriegn key of the parent model on the child model

            parent param is the instance of the parent model

            data is the list of dicts for the validated_data in the parent serializer
        """
        meta = self.child.Meta
        for i in data:
            if hasattr(meta, 'from_parent_fields'):
                for field in meta.from_parent_fields:
                    if hasattr(parent, field):
                        i[field] = getattr(parent, field, None)
            if hasattr(meta, 'generic_foreign_key'):
                i[meta.generic_foreign_key[0]] = parent.pk
                i[meta.generic_foreign_key[1]] = ContentType.objects.get_for_model(parent)
            elif parent_field:
                i[parent_field] = parent

    def update(self, instance, validated_data):

        # Maps for id->instance and id->data item.
        obj_mapping = {obj.pk: obj for obj in instance}

        existing_pks = []

        ret = []

        # find existing pks
        for data in validated_data:
            if data.get('id', None):
                pk = data['id']
                existing_pks.append(pk)

        # Perform deletion of missing pks
        self.child.Meta.model.objects.filter(id__in=set(obj_mapping.keys()) - set(existing_pks)).delete()

        for data in validated_data:
            if data.get('id', None):
                # perform update
                pk = data['id']
                obj = obj_mapping[pk]
                ret.append(self.child.update(obj, data))
            else:
                # perform create
                ret.append(self.child.create(data))

        return ret
```

In the above code the `set_parent_data` is used to set data from the parent object on the child object by adding it the data dicts. This is done so that the API consumer does not have to supply the same data again and again in the list of child data. `set_parent_data` is called in the parent serialiser create and update method to set the values in the child data before saving the child data.

`from_parent_fields` is set on the meta class of the child serializer class. This is a list of fields which are copied from the parent instance to the child instances.

`foreign_key_field_name` is set on the meta class of the child serializer as the name of the foreign key field of the parent model on the child model. For example the child model `line_item` has foreign key field `invoice` on it, so the value of `foreign_key_field_name` will be `invoice`.

To the use the above list serializer class you have to set it on the child serialiser class as the `list_serializer_class` in the meta class of the child serializer.

Below is an example of usage of the above serializer classes. The parent model is `PurchaseOrder` and the child model is the purchase order line item called  `Poline`.

```
class PurchaseOrderSerializer(NestedSerializerMixin):
    lines = PoLineSerializer(many=True, required=False)
    class Meta:
        model = PurchaseOrder
        fields = '__all__'


class PoLineSerializer(serializers.ModelSerializer):
    id = serializers.IntegerField(required=False)

    class Meta:
        model = PoLine
        fields = '__all__'
        list_serializer_class = ListUpdateSerializer
        from_parent_fields = ['client_id', 'contact_id', 'warehouse_id', 'currency']
        foreign_key_field_name = 'purchase_order'
        # set by the serializer using parent instance so marked as read only
        read_only_fields = from_parent_fields + [foreign_key_field_name,]
```