Title: Django admin: Inline в Fieldset'і або як впихнути невпихуєме
Date: 13-02-2017
Slug: django-inline-in-fieldset
Category: Django
Lang: uk
Image: /images/django-inline-in-fieldset-with-css.png
Featured_image: /images/django-inline-in-fieldset-with-css.png
Tags: django, python, admin, StackOverFlow-driven development
Summary: 


<p class="text-right"><q>Вони хочуть впихнути невпихуєме</q><br>&mdash; Іван Плющ</p>

Позиціонування TabularInline, здавалось доволі просте завдання з яким починають стикатись Django
програмісти коли розмір ModelAdmin форми починає перевалювати за декілька "екранів" і порядок полів
стає критично важливим для орієнтування, але на диво в Django немає стандарного механізму для
вирішення цього завдання.

## Проблема

Для прикладу створимо "сферичний додаток у вакуумі" `sample_app` з двома моделями: `ModelA` та `Image`.

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

А тепер давайте спробуємо створити простенький Tabular Inline і розмістити елементи на формі у наступному порядку:

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

Як можна здогадатись у нас нічого не вийде і натомість ми отримаємо помилку:

```
TypeError at /admin/sample_app/modela/add/
sequence item 0: expected str instance, MediaDefiningClass found
....
```

## Вирішення

У всіх варіаціях цієї проблеми на StackOverFlow фігурує абстрактна відповідь: "змінюйте
`change_form.html`", і чомусь ніде я не знайшов готового сніпету тому ось вам власний.

__admin.py__

Для початку додамо змінну класу (class variable) `insert_after` до `ImageInline`, і вкажемо, що цей
inline ми хочемо бачити після поля title, сам `ImageInline` помістимо у відповідний список.

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

Тепер створимо власний шаблон для форми редагування на
основі
[базового шаблону](https://github.com/django/django/blob/master/django/contrib/admin/templates/admin/change_form.html). Нас
цікавлять цикли які відповідають за fieldset'и і за вивід inline'ів:

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

{# Для того щоб уникнути повторного виведення inline'ів кінці форми фільтруємо їх в циклі #}
{% block inline_field_sets %}
    {% for inline_admin_formset in inline_admin_formsets %}
        {% if not inline_admin_formset.opts.insert_after %}
            {% include inline_admin_formset.opts.template %}
        {% endif %}
    {% endfor %}
{% endblock %}
```

І оновлюємо шаблон для fieldset'у який і рендеритиме інлайни:

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
                {# Вставляємо inline'и після поля #}
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

Після цього ми можемо "насолоджуватись" наступною картиною:

![Django TabularInline in fieldset](images/django-inline-in-fieldset.png)

Залишилось трішки підправити CSS, щоб при вигляді нашої адмінки у перфекціоністів не сіпалось око:

__static/css/admin.css__

```
:::css

.form-row .inline-group {
    margin-top: 30px;
}
```

## Результат

<p style="margin-bottom: 0">Разом з кінцевим результатом додаю посилання на демо-проект який можна "помацати руками".</p>
<div class="col-xs-12 col-sm-12 text-center" style="margin-bottom:10px;">
<a class="btn btn-success btn-raised btn-lg" 
href="https://github.com/linevich/samples/tree/master/inlines_in_fieldsets" target="_blabk">
Проект на Github<i class="material-icons">open_in_new</i>
</a>
</div>

![Inline вставлений в fieldset. Django admin.](images/django-inline-in-fieldset-with-css.png)

Якщо я зекономив вам трохи часу, будь ласка, поділіться цим постом в Twitter/Facebook або будь-якій
іншій соціальній мережі, дякую.
