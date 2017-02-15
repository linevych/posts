Title: Django admin: Inline in Fieldset
Date: 13-02-2017
Slug: django-inline-in-fieldset
Category: Django
Lang: en
Image: /images/django-inline-in-fieldset-with-css.png
Featured_image: /images/django-inline-in-fieldset-with-css.png
Tags: django, python, admin, StackOverflow-driven development
Summary: 


The positioning of the TabularInline, it would seem to be a simple task. Most of Django developers
face this challenge when the size of ModelAdmin becomes bigger than the several "screens" and
ordering becomes critical for the usability.

## Problem

For example let's create an app called `sample_app` with two models: `ModelA` and `Image`.

__models.py__

```
:::python 

from django.db import models
from django.utils.translation import ugettext_lazy as _


class ModelA(models.Model):
    title = models.CharField(
        verbose_name=_('title'),
        max_length=255,
        blank=True,
        null=True,
    )

    description = models.TextField(
        verbose_name=_('Description'),
        blank=True,
        null=True,
    )

    def __str__(self):
        return self.title


class Image(models.Model):
    model_a = models.ForeignKey(
        'ModelA',
        verbose_name='model A'
    )
    image = models.ImageField(
        upload_to='images/'
    )

```

Now let's create simple TabularInline and try to insert this in the following order:

1. `title`
2. `ImageInline`
3. `description`

__admin.py__

```
:::python

from django.contrib import admin

from .models import ModelA, Image


class ImageInline(admin.TabularInline):
    model = Image


class ModelAAdmin(admin.ModelAdmin):
    fields = (
        'title',
        ImageInline,
        'description'
    )

admin.site.register(ModelA, ModelAAdmin)

```
As you can guess we can't do this and will get an error:

```
TypeError at /admin/sample_app/modela/add/
sequence item 0: expected str instance, MediaDefiningClass found
....
```

## Solution

In all variations of this problem at StackOverflow, we can find an abstract answer like: "customize
`change_form.html`", but for some reason, I can't find any of the ready to use snippets, so here is my own.

__admin.py__

To start let's define the class variable `insert_after` in the `ImageInline`, and set that we want to see that
inline after the field "title" and put `ImageInline` to the appropriate list.

```
:::python

from django.contrib import admin

from .models import ModelA, Image


class ImageInline(admin.TabularInline):
    model = Image
    insert_after = 'title'


class ModelAAdmin(admin.ModelAdmin):
    fields = (
        'title',
        'description'
    )
    inlines = [
        ImageInline,
    ]
    change_form_template = 'admin/custom/change_form.html'

    class Media:
        css = {
            'all': (
                'css/admin.css',
            )
        }


admin.site.register(ModelA, ModelAAdmin)
```

Now let's create our own template for the editing form using
the
[basic template](https://github.com/django/django/blob/master/django/contrib/admin/templates/admin/change_form.html). We
need to customize cycles that are responsible for the rendering inlines:

__templates/admin/custom/change_form.html__


```
:::django

{% extends "admin/change_form.html" %}
{% load i18n admin_urls static admin_modify %}
{% block field_sets %}
    {% for fieldset in adminform %}
        {#  Власний шаблон для виводу fieldset'ів #}
        {% include "admin/custom/fieldset.html" with inline_admin_formsets=inline_admin_formsets %}
    {% endfor %}
{% endblock %}

{# Filter inlines that where already rendered to avoid duplication #}
{% block inline_field_sets %}
    {% for inline_admin_formset in inline_admin_formsets %}
        {% if not inline_admin_formset.opts.insert_after %}
            {% include inline_admin_formset.opts.template %}
        {% endif %}
    {% endfor %}
{% endblock %}
```

And update the fieldsets template which will be rendering the inlines.

__templates/admin/custom/fieldset.html__

```
:::django

<fieldset class="module aligned {{ fieldset.classes }}">
    {% if fieldset.name %}<h2>{{ fieldset.name }}</h2>{% endif %}
    {% if fieldset.description %}
        <div class="description">{{ fieldset.description|safe }}</div>
    {% endif %}
    {% for line in fieldset %}
        <div class="form-row{% if line.fields|length_is:'1' and line.errors %} errors{% endif %}{% if not line.has_visible_field %} hidden{% endif %}{% for field in line %}{% if field.field.name %} field-{{ field.field.name }}{% endif %}{% endfor %}">
            {% if line.fields|length_is:'1' %}{{ line.errors }}{% endif %}
            {% for field in line %}
                <div{% if not line.fields|length_is:'1' %}
                    class="field-box{% if field.field.name %} field-{{ field.field.name }}{% endif %}{% if not field.is_readonly and field.errors %} errors{% endif %}{% if field.field.is_hidden %} hidden{% endif %}"{% elif field.is_checkbox %}
                    class="checkbox-row"{% endif %}>
                    {% if not line.fields|length_is:'1' and not field.is_readonly %}{{ field.errors }}{% endif %}
                    {% if field.is_checkbox %}
                        {{ field.field }}{{ field.label_tag }}
                    {% else %}
                        {{ field.label_tag }}
                        {% if field.is_readonly %}
                            <div class="readonly">{{ field.contents }}</div>
                        {% else %}
                            {{ field.field }}
                        {% endif %}
                    {% endif %}
                    {% if field.field.help_text %}
                        <div class="help">{{ field.field.help_text|safe }}</div>
                    {% endif %}
                </div>
                {# Insert inlines after the field #}
                {% for inline_admin_formset in inline_admin_formsets %}
                        {% if inline_admin_formset.opts.insert_after == field.field.name %}
                            {% include inline_admin_formset.opts.template %}
                    {% endif %}
                {% endfor %}
            {% endfor %}
        </div>
    {% endfor %}
</fieldset>
```

After that we can "enjoy" of the following:

![Django TabularInline in fieldset](/images/django-inline-in-fieldset.png)

Still only need to fix some CSS, to protect the perfectionists from an eye bleeding:

__static/css/admin.css__

```
:::css

.form-row .inline-group {
    margin-top: 30px;
}
```

## Result

<p style="margin-bottom: 0">Here is the final result of our work and the whole project that you can
"touch" on the Github.</p>
<div class="col-xs-12 col-sm-12 text-center" style="margin-bottom:10px;">
<a class="btn btn-success btn-raised btn-lg" href="https://github.com/linevich/samples/tree/master/inlines_in_fieldsets"
target="_blabk">
Project on the Github<i class="material-icons">open_in_new</i>
</a>
</div>

![The inline at the fieldset. Django Admin](/images/django-inline-in-fieldset-with-css.png)

English is not my mother tongue so if you have found any mistakes or know how to improve this text
please contact me directly.

If I've saved a little of your time please share this post on the social networks. Thanks!
