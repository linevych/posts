Title: Перекладаємо моделі в Django за допомогою django-modeltranslation
Date: 2015-01-22 01:29
Category: Django
Tags: django, переклади, інтернаціоналізація, python
Slug: django-models-translating
Image: /images/django.jpg
Lang: uk

Під час роботи над одним із проектів мені знадобилась підтримка декількох мов, в якості інструменту було вибрано
[django-modeltranslation](https://github.com/deschler/django-modeltranslation), через легкість до інтеграції в існуючий код.

## Процес встановлення
Встановлюємо пакунок:

    pip install django-modeltranslation

Підключаємо його у список встановлених:

    :::python
    INSTALLED_APPS = (
        ......
        'modeltranslation',
        ......
    )

Прописуємо налаштування додатку в `settings.py`
    
    :::python
    MODELTRANSLATION_DEFAULT_LANGUAGE = 'uk' # Мова для перекладу за замовчуванням

    # Ця лямда-функція необхідна для того, щоб Django міг перекладати verbose_name
    # в моделях використовуючи для цього вбудоване рішення на базі i18n
    gettext = lambda s: s

    # Всі мови на які можливо буде перекласти контент
    LANGUAGES = (
        ('uk', gettext('Ukrainian')),
        ('ru', gettext('Russian')),
        ('en', gettext('English')),
    )

    MODELTRANSLATION_FALLBACK_LANGUAGES = ('uk', 'ru') # Альтернативні(запасні) переклади 

    # Відлагоджування для production повинно бути вимкнене, але для розробки допоможе
    MODELTRANSLATION_DEBUG = True 

Тепер для прикладу приведу частину моделі з додатку (в моєму випадку це `blog`):

    :::python
    class Post(models.Model):
        """
        Default post model
        """
        title = models.CharField(
            verbose_name=u"Назва",
            max_length=100,
            blank=True,
            null=True,
            default=u"Untitled",
        )
        text = models.TextField(
            verbose_name=u"Текст",
            blank=True,
            null=True,
        )
        preview_text = models.TextField(
            verbose_name=u"Текст попереднього перегляду",
            blank=True,
            null=True,
        )
        .......

Тепер реєструємо нашу модель для перекладу для цього створюємо файл `blog/translation.py`

    :::python
    from modeltranslation.translator import translator, TranslationOptions
    from blog.models import Post


    class PostTranslationOptions(TranslationOptions):
        # Вказуємо поля які потребують перекладу
        fields = ('title', 'text', 'preview_text',)


    translator.register(Post, PostTranslationOptions)

І підключаємо модель в адмінку:

    :::python
    # Фіксимо баг, про який буде йти мова далі:
    import blog.translation
    from modeltranslation.admin import TranslationAdmin
    from django.contrib import admin
    from blog.models import Post


    class PostAdmin(TranslationAdmin):
        pass


    admin.site.register(Post, PostAdmin)

Для того щоб додати наші поля перекладу в базу виконуємо:

    python manage.py sync_translation_fields 

## Граблі
Зробивши все як написано в документації я отримав помилку:

    modeltranslation.translator.NotRegistered: The model "ModelName" is not registered for translation

Швидкий Googl'інг дав зрозуміти, що я [не перший стикнувся з цією проблемою](http://djbook.ru/forum/topic/2605/).
Рішення було доволі простим імпортувати `translation.py`безспосередньо в `admin.py:`

    import blog.translation

## Результат

![djnago-modeltranslation](/images/django-modeltranslations-screenshot.png)

