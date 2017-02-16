Title: How to get list of all fields from Django ModelForm
Date: 2017-02-16
Category: Django
Tags: Django, python
Slug: get-fields-from-django-modelform
Image: /images/django.jpg
Lang: en
Summary: How to get a list of all fields from Django ModelForm (ready to use solution).

Somehow I had to cover the form with tests, all fields in TestCase should be set manually and
updated by hands with every model change. I wanted to avoid this routine, solution should be simple
--- grab the list of fields from the ModelForm and iterate over it. But there is a problem: by
default, there is no way to get a list of fields considering that some of them are excluded or added
to the form manually.

## Solution

As a result of a short digging into the code of a ModelForm, I wrote the simple function that does
exactly what I need, so I would like to share it here. Maybe it saves you a little time.

```
:::python
def get_all_fields_from_form(instance):
    """"
    Return names of all available fields from given Form instance.

    :arg instance: Form instance
    :returns list of field names
    :rtype: list
    """

    fields = []
    for item in instance.__dict__.items():
        if item[0] == 'declared_fields' or item[0] == 'base_fields':
            for field in item[1]:
                if hasattr(instance.Meta, 'exclude'):
                    if field not in instance.Meta.exclude:
                        fields.append(field)
                else:
                    fields.append(field)
    return fields
```

## Improved solution

After I've posted this article on the Reddit, I've got
the
[comment](https://www.reddit.com/r/django/comments/5ubix3/how_to_get_list_of_all_fields_from_django/ddsvp68/) from
[Nicksil](https://www.reddit.com/user/Nicksil) that makes me feel stupid, the function can be
implemented much simpler:

```
:::python

def get_all_fields_from_form(instance):
    """"
    Return names of all available fields from given Form instance.

    :arg instance: Form instance
    :returns list of field names
    :rtype: list
    """

    fields = list(instance().base_fields)

    for field in list(instance().declared_fields):
        if field not in fields:
            fields.append(field)
    return fields


```

That's it!


## Usage

```
:::python

# Let's create a simple model for the example
class ModelName(models.Model):
    field1 = models.DummyField()
    field2 = models.DummyField()
    field3 = models.DummyField()
    field4 = models.DummyField()


# And form for our model
class MyForm(ModelForm):
    class Meta:
       model = ModelName
       exclude = [field1, field2]


# Let's print-out the list of fields:
print(get_all_fields_from_form(MyForm))

# Output:
['field3', 'field4']
```

