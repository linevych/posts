Title: Improving Django-Select2 Widget with Annotation's magic.
Date: 2017-02-27
Category: Django
Tags: django, python, annotation, select2, queryset
Slug: django-select2-annotation-magic
Image: /images/django.jpg
Lang: en
Summary: How to get a list of all fields from Django ModelForm (ready to use solution).


In this post I will create widget to select users that allows to filter by username, first name and
last name and display it in readable
format:

![Django Select2: showing user first_name and last_name after login](/images/django-select2-user-widget.gif)

## Example project

### Model

Let's create model `Event` to store events, of course we need to store title of the event, few
details about it people who are inveted to this event and person that organize it. Our model will
looks like follwing:

```
:::python

from django.db import models
from django.contrib.auth import get_user_model
from django.utils.translation import ugettext_lazy as _

USER_MODEL = get_user_model()


class Event(models.Model):
    title = models.CharField(
        verbose_name=_('title'),
        max_length=255,
        blank=True,
        null=True,
    )
    description = models.TextField(
        verbose_name=_('description'),
        blank=True,
        null=True,
    )
    creator = models.ForeignKey(
        USER_MODEL,
        blank=False,

    )
    members = models.ManyToManyField(
        USER_MODEL,
        blank=True
    )

    def __str__(self):
        return self.title

```

### Admin panel

Now let's create admin panel for our model, but here is one thing: if we have a lot of users in the
databse it will be very difficult to select one from a huge list. For this purpose we can user
`ModelSelect2Widget` for `ForeignKey` field and `ModelSelect2MultipleWidget for `ManyToManyfield`.

```
:::python
```

Looks fine, but 

## Custom widget and some magic

## Bonuses
