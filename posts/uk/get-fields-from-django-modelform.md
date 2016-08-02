Title: Витягаємо список полів з ModelForm в Django
Date: 2016-06-22 22:00
Category: Django
Tags: django, python
Slug: get-fields-from-django-modelform
Image: /images/django.jpg
Lang: uk
Summary: Як просто отримати список всіх доступних полів з ModelForm.

Довелось мені покривати тестами форму в проекті, всі поля в тестах звісно задавались руками і
хотілось би якось позбавитись рутинної правки тесту, щоразу коли додаєш звичайне поле.  Рішення було
знайдено одразу: видрати з ModelForm'и fields і на основі цього списку будувати свій тест, але все
виявилось не так просто як здавалось. За замовчуванням отримати список полів з урахуванням
виключених і доданих безпосередньо в формі на пряму не можна (читай: я не знайшов як то зробити).

## Рішення

В результаті нетривалого дослідження нутрощів ModelForm'и намалювався така собі примітивна функція,
яку я залишу на загальний огляд тут в цьому пості. Можливо вона комусь зекономить трохи часу.

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

## Використання

```
:::python

# Створимо модельку для прикладу
class ModelName(models.Model):
    field1 = models.DummyField()
	field2 = models.DummyField()
    field3 = models.DummyField()
    field4 = models.DummyField()


# І форму для неї:
class MyForm(ModelForm):
	class Meta:
	   model = ModelName
	   exclude = [field1, field2]


# Друкуємо список полів:
print(get_all_fields_from_form(MyForm))

# Вивід:
['field3', 'field4']
```

## P.S

Все таки чомусь мені здається, що я трохи намудрив, можливо ви знаєте кращий метод, напишіть про це
в коментарях.
