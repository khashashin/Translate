[Документация](http://docs.wagtail.io/en/v1.13.1/index.html) > [Руководство по использованию](http://docs.wagtail.io/en/v1.13.1/topics/index.html) > Фрагменты ///////////////////// [Редактировать на GitHub](https://github.com/wagtail/wagtail/blob/b0be9edf510ce414c4050195817094b9261bc945/docs/topics/permissions.rst)
___

**Фрагменты**
----------
Отрывки - части контента, которые не требуют полного построения веб-страницы для рендеринга. Они могут использоваться для создания вторичного контента, такого как заголовки, нижние колонтитулы и боковые панели, редактируемые в администраторе Wagtail. Фрагменты - это модели Django, которые не наследуют класс `Page` и поэтому они не организованы в структуру Wagtail. Тем не менее, они могут быть редактированы, назначая панели и идентифицируя модель как фрагмент с помощью декоратора класса `register_snippet`.

У фрагментов недостает многих функций страниц, например, возможность упорядочивание в администраторе Wagtail или иметь определенный URL-адрес. Тщательно обдумайте, когда вы будите строить контент с помощью фрагмента. Фрагмент может быть более подходящим для страницы.

**Snippet Models**
--------------
Вот пример фрагмента с демонстрационного веб-сайта Wagtail:

>     from django.db import models
>     from django.utils.encoding import python_2_unicode_compatible
>     
>     from wagtail.wagtailadmin.edit_handlers import FieldPanel
>     from wagtail.wagtailsnippets.models import register_snippet
>     
>     ...
>     
>     @register_snippet
>     @python_2_unicode_compatible  # provide equivalent __unicode__ and __str__ methods on Python 2
>     class Advert(models.Model):
>         url = models.URLField(null=True, blank=True)
>         text = models.CharField(max_length=255)
>     
>         panels = [
>             FieldPanel('url'),
>             FieldPanel('text'),
>         ]
>     
>         def __str__(self):
>             return self.text

Модель `Advert` использует базовый класс модели Django и определяет два свойства: текст и URL. Интерфейс редактирования очень похож на тот, что предоставляется для `Page` - производные модели с полями прописываются в свойстве `panels`. Фрагменты не используют несколько вкладок полей, и они не обеспечивают функции «сохранить как черновик» или «представить для модерации» в Админ-панели.

`@register_snippet` сообщает Wagtail рассматривать модель как фрагмент. Список `panels` определяет поля, отображаемые на странице редактирования фрагмента. Также важно предоставить строковое представление класса через `def __str __ (self):` так чтобы объекты фрагмента имели смысл, когда они перечисляются в администраторе Wagtail.

**Включение фрагментов в теги шаблонов**
------------------------------------
Самый простой способ сделать ваши фрагменты доступными для шаблонов - это тег шаблона. Это в основном делается с чистого Django, поэтому рассмотрение [документации Django](https://docs.djangoproject.com/en/dev/howto/custom-template-tags/) для пользовательских тегов шаблонов будет более полезным. Однако мы рассмотрим основы и отметим любые соображения, которые нужно сделать для Wagtail.

Сначала добавьте новый файл python в папку `templatetags` в вашем приложении. Демо-сайт, например, использует путь `wagtaildemo/demo/templatetags/demo_tags.py`. Нам нужно будет загрузить некоторые модули Django и наши модели приложений и добавить декоратор `register`:

>     from django import template
>     from demo.models import *
>     
>     register = template.Library()
>     
>     ...
>     
>     # Advert snippets
>     @register.inclusion_tag('demo/tags/adverts.html', takes_context=True)
>     def adverts(context):
>         return {
>             'adverts': Advert.objects.all(),
>             'request': context['request'],
>         }

`@register.inclusion_tag()` принимает две переменные: шаблон и boolean о том, должен ли этот шаблон передавать контекст запроса. Рекомендуется включать контексты запросов в ваши пользовательские теги шаблонов, так как некоторые теги шаблонов Wagtail, такие как `pageurl` требуют контекст чтобы они правильно работали. The template tag function could take arguments and filter the adverts to return a specific model, but for brevity we’ll just use Advert.objects.all().

Вот что находится в шаблоне, используемом этим тегом шаблона:

>     {% for advert in adverts %}
>         <p>
>             <a href="{{ advert.url }}">
>                 {{ advert.text }}
>             </a>
>         </p>
>     {% endfor %}

Затем в ваших собственных шаблонах страниц вы можете включить этот тег:

>     {% load wagtailcore_tags demo_tags %}
>     
>     ...
>     
>     {% block content %}
>     
>         ...
>     
>         {% adverts %}
>     
>     {% endblock %}

**Связывание страниц с фрагментами**
--------------------------------
В приведенном выше примере, это фиксированный список объявлений, который отображается независимо от содержимого страницы. Это могло бы быть то, что вы хотите для общей боковой панели, но в некоторых случаях вы можете обратиться к определенному фрагменту из содержимого страницы. Это можно сделать, определив внешний ключ к модели фрагмента в вашей модели страницы, и добавление `SnippetChooserPanel` в список `content_panels` страницы. Например, если вы хотите указать объявление, которое должно появиться в `BookPage`:
from wagtail.wagtailsnippets.edit_handlers import SnippetChooserPanel

>     # ...
>     class BookPage(Page):
>         advert = models.ForeignKey(
>             'demo.Advert',
>             null=True,
>             blank=True,
>             on_delete=models.SET_NULL,
>             related_name='+'
>         )
>     
>         content_panels = Page.content_panels + [
>             SnippetChooserPanel('advert'),
>             # ...
>         ]

После этого фрагмент можно получить в вашем шаблоне в виде `page.advert`.

Чтобы прикрепить несколько объявлений к странице, `SnippetChooserPanel` можно поместить на встроенный дочерний объект `BookPage`, а не на самой `BookPage`. Эта дочерняя модель называется `BookPageAdvertPlacement` (so called because there is one such object for each time that an advert is placed on a BookPage):

>     from django.db import models
>     
>     from wagtail.wagtailcore.models import Page, Orderable
>     from wagtail.wagtailsnippets.edit_handlers import SnippetChooserPanel
>     
>     from modelcluster.fields import ParentalKey
>     
>     ...
>     
>     class BookPageAdvertPlacement(Orderable, models.Model):
>         page = ParentalKey('demo.BookPage', related_name='advert_placements')
>         advert = models.ForeignKey('demo.Advert', related_name='+')
>     
>         class Meta:
>             verbose_name = "advert placement"
>             verbose_name_plural = "advert placements"
>     
>         panels = [
>             SnippetChooserPanel('advert'),
>         ]
>     
>         def __str__(self):
>             return self.page.title + " -> " + self.advert.text
>     
>     
>     class BookPage(Page):
>         ...
>     
>         content_panels = Page.content_panels + [
>             InlinePanel('advert_placements', label="Adverts"),
>             # ...
>         ]

Эти дочерние объекты теперь доступны через свойство страницы `advert_placements`,  и отсюда мы можем получить доступ ко всем связанным объектам модели через `advert`. В шаблоне для `BookPage` мы можем включить следующее:

>     {% for advert_placement in page.advert_placements.all %}
>         <p>
>             <a href="{{ advert_placement.advert.url }}">
>                 {{ advert_placement.advert.text }}
>             </a>
>         </p>
>     {% endfor %}

**Создание фрагментов доступных для поиска**
----------------------------------------
Если модель фрагмента наследуется от `wagtail.wagtailsearch.index.Indexed`, как описано в [пользовательских моделях индексирования](http://docs.wagtail.io/en/v1.13.1/topics/search/indexing.html#wagtailsearch-indexing-models), Wagtail автоматически добавит окно поиска в интерфейс выбора для этого типа фрагмента. Например, фрагмент `Advert`  может быть сделана для поиска следующим образом:

>     ...
>     
>     from wagtail.wagtailsearch import index
>     
>     ...
>     
>     @register_snippet
>     class Advert(index.Indexed, models.Model):
>         url = models.URLField(null=True, blank=True)
>         text = models.CharField(max_length=255)
>     
>         panels = [
>             FieldPanel('url'),
>             FieldPanel('text'),
>         ]
>     
>         search_fields = [
>             index.SearchField('text', partial_match=True),
>         ]

**Маркировка фрагментов**
------------------
Добавление тегов к фрагментам очень похоже на добавление тегов к страницам. Единственное различие заключается в том, что вместо `ClusterTaggableManager` следует использовать  `taggit.manager.TaggableManager`.

>     from modelcluster.fields import ParentalKey
>     from modelcluster.models import ClusterableModel
>     from taggit.models import TaggedItemBase
>     from taggit.managers import TaggableManager
>     
>     class AdvertTag(TaggedItemBase):
>         content_object = ParentalKey('demo.Advert', related_name='tagged_items')
>     
>     @register_snippet
>     class Advert(ClusterableModel):
>         ...
>         tags = TaggableManager(through=AdvertTag, blank=True)
>     
>         panels = [
>             ...
>             FieldPanel('tags'),
>         ]

В [документации по маркировкам страниц](http://docs.wagtail.io/en/v1.13.1/reference/pages/model_recipes.html#tagging) содержится дополнительная информация о том, как использовать теги в представлениях(views).
