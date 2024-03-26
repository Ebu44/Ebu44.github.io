---
title: Django ArrayField
author: Ebu
date: 2023-06-20 16:00:00 +0800
categories: [Python, Django]
tags: [Python, Django, ArrayField]
pin: true
---

## Django ArrayField

Hello There!

In Django, Arrayfield would be confusing. Despite the complexity of ArrayFields, I couldn’t find a comprehensive and explanatory resource with sample examples. That’s why I decided to write this blog. Today, we will talk about ArrayFields.

However, I should note beforehand that ArrayField is a feature that is unfortunately only applicable in PostgreSQL. But don’t worry, because there are other solutions available for different scenarios.

Alternatively for other databases you can use JSONField or ManyToManyField.

Then let’s get started.

As mentioned in documentation of ArrayField: A field for storing lists of data.

Instead of creating another table you can use Arrayfield like:
tags, features, options etc.

Also ArrayField can be nested to store multi-dimensional arrays.
For example:
```
multi_dimensional = ArrayField(ArrayField(models.IntegerField()))
```

```
# class ArrayField(base_field, size=None, **options)

# For example if you want to store list of CharField:
tags = ArrayField(models.CharField(max_length=200), blank=True)

# or IntegerField:
pieces = ArrayField(models.IntegerField())
```

as you can see, it is easy and clear.

## How Can You Use ArrayField?

First of all you have to import ArrayField

```
from django.contrib.postgres.fields import ArrayField
```

Now we can code our model class

```
class GamerList(models.Model):
    name = models.CharField(max_length=128)
    age = models.IntegerField()
    tags = ArrayField(
        models.CharField(max_length=512)
    )
```

As you can see i create tags ArrayField with CharField. Let’s add some data

```
GamerList.objects.create(name="Ebu", age=23, tags=["total war", "django"])
```

then let’s see what happens

```
def arrayfield_view(request):
    list_of_gamers = GamerList.objects.get(id=1)
    print(f"name: {list_of_gamers.name} \nage: {list_of_gamers.age} \ntags: {list_of_gamers.tags}")
```
```
name: Ebu 
age: 23
tags: ['total war', 'django']
```

As you can see our tags field returned array with 2 elements that we added earlier.

## Querying ArrayField

Ok so how we can query this data?

There are lots of custom lookups. But for now i wanna show you 2 of them:

contains:
```
list_of_gamers = GamerList.objects.filter(tags__contains=['django'])
# <QuerySet [<GamerList: Ebu - ['total war', 'django']>]>

list_of_gamers = GamerList.objects.filter(tags__contains=['not exist'])
# <QuerySet []>
```
len:
```
list_of_gamers = GamerList.objects.filter(tags__len=1)
#<QuerySet []>

list_of_gamers = GamerList.objects.filter(tags__len=2)
# <QuerySet [<GamerList: Ebu - ['total war', 'django']>]>
```

You can check documentation for more lookups: https://docs.djangoproject.com/en/4.2/ref/models/querysets/#field-lookups

## Counting distinct elements in Django ArrayField

Let’s explore even more frightening scenarios.

if you want to count distinct elements in Django ArrayField you can do it like that:
```
list_of_gamers = GamerList.objects.annotate(tag=Func(F('tags'), function='unnest')).
                                   values('tag').
                                   order_by('tag').
                                   annotate(count=Count('id')).
                                   values_list('tag', 'count')
```
```
# <QuerySet [('django', 2), ('Java', 1), ('javascript', 1), ('python', 1), ('total war', 2)]>
```
GamerList.objects.annotate(tag=Func(F('tags'), function='unnest')) initializes a query on the GamerList model. The annotate method is used to add a new field for each object. In this case, a field named tag is added. The Func class allows you to use database functions. The expression F('tags') is used to operate on the tags field. The parameter function='unnest' transforms the tags field into a separate column.

.values('tag') performs a query that only returns the tag field. This indicates that the query is only interested in the tag field.

.order_by('tag') ensures that the results are sorted by the tag field. In this case, the results are sorted alphabetically based on the tag field.

.annotate(count=Count('id')) adds a field named count. The expression Count('id') calculates the number of matching objects for each tag and assigns it to the count field.

.values_list('tag', 'count') returns a list containing the tag and count fields. This means that each element in the result is a tuple containing the tag and count fields.

Yeah, I know it’s confusing, but believe me, it’s worth it.

## Conclusion

In conclusion, the Django ArrayField provides a convenient way to store lists of data within a model. Despite its initial complexity, finding comprehensive resources on ArrayFields can be challenging. This blog aimed to address that gap and shed light on working with ArrayFields.

However, it’s important to note that ArrayField is only applicable in PostgreSQL and not supported in other databases. Nonetheless, there are alternative solutions available for different scenarios, such as using JSONField or ManyToManyField.

When querying ArrayField, Django provides various custom lookups. Two common examples are using the contains lookup to filter based on array elements and the len lookup to count the length of the array.

Although ArrayFields might seem confusing at first, the flexibility they provide is valuable and worth exploring.
