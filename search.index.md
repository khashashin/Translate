[Документация](http://docs.wagtail.io/en/v1.13.1/index.html) > [Руководство по использованию](http://docs.wagtail.io/en/v1.13.1/topics/index.html) > [Поиск](http://docs.wagtail.io/en/v1.13.1/topics/search/index.html) > Индексация ///////// [Редактировать на GitHub](https://github.com/wagtail/wagtail/blob/b0be9edf510ce414c4050195817094b9261bc945/docs/topics/permissions.rst)
___

**Индексирование**
--------------
Чтобы сделать модель доступной для поиска, вам нужно добавить ее в индекс поиска. Все страницы, изображения и документы индексируются так и так, поэтому вы можете сразу производить поиск по ним.

Если вы создали дополнительные поля в подклассе Page или Image, вы можете также добавить эти новые поля в индексацию поиска, чтобы контент из этих полей выдавался по поисковому запросу пользователя. См. [дополнительные поля индексирования](http://docs.wagtail.io/en/v1.13.1/topics/search/indexing.html#wagtailsearch-indexing-fields), для получения информации о том, как это сделать.

Если у вас есть пользовательская модель, которую вы хотите сделать доступной для поиска, см. [Индексирование пользовательских моделей](http://docs.wagtail.io/en/v1.13.1/topics/search/indexing.html#wagtailsearch-indexing-models).

**Обновление индекса**
------------------
Если индексация поиска хранится отдельно от  основной базы данных (например, при использовании Elasticsearch), вам нужно синхронизировать ваши базы. Есть два способа сделать это: используя обработчики сигналов поиска, или периодически вызывать команду `update_index`. Для обеспечения максимальной скорости и надежности, лучше использовать оба варианта, если это возможно.

**Обработчики сигналов**
--------------------
Изменено в версии 0.8: Обработчики сигналов теперь автоматически регистрируются

`wagtailsearch` предоставляет некоторые обработчики сигналов, которые подключаются к сигналами сохранения/удаления всех индексированных моделей. Он будет автоматически добавлять и удалять их из всех баз данных, которые вы зарегистрировали в настройках `WAGTAILSEARCH_BACKENDS`.
Эти обработчики сигналов автоматически регистрируются при загрузке приложения `wagtail.wagtailsearch`.

**Команда `update_index`**
--------------------
Wagtail также предоставляет команду для реконструкцию индекса с нуля.

`./manage.py update_index`

Рекомендуется запускать эту команду один раз в неделю и в следующие моменты:

 - всякий раз, когда какие-либо страницы создаются с помощью скрипта (после импорта, например)
 - когда любые изменения были внесены в модели или в конфигурацию поиска

Поиск может не возвращать никаких результатов во время выполнения этой команды, поэтому избегайте его запуска в пиковые времена.

**Индексирование дополнительных полей**
-----------------------------------
**Предупреждение**
Indexing extra fields is not supported by the database backend. If you’re using the database backend, any other fields you define via `search_fields` will be ignored.
Индексирование дополнительных полей не поддерживается серверной базой данных. Если вы используете серверную базу данных, любые другие поля, которые вы определяете через `search_fields`, будут игнорироваться.

Поля должны быть явно добавлены к свойству `search_fields` в вашего модели построенную на `Page`, чтобы вы могли их искать или фильтровать. Это делается путем переписывания `search_fields`, чтобы добавить к нему список дополнительных объектов `SearchField` / `FilterField`.

**Пример**
------
Это создает модель `EventPage` с двумя полями: `description` и `date`. `description` индексируется как SearchField а `date`  индексируется как FilterField

>     from wagtail.wagtailsearch import index
>     from django.utils import timezone
>     
>     class EventPage(Page):
>         description = models.TextField()
>         date = models.DateField()
>     
>         search_fields = Page.search_fields + [ # Inherit search_fields from Page
>             index.SearchField('description'),
>             index.FilterField('date'),
>         ]
>     
>     
>     # Get future events which contain the string "Christmas" in the title or description
>     >>> EventPage.objects.filter(date__gt=timezone.now()).search("Christmas")

**`index.SearchField`** - Это используются для выполнения полнотекстовых поисков по вашим моделям, как правило, для текстовых полей.

**Опции**
-----

 - **partial_match**  (`boolean`) - Установка этого значения в true позволяет сопоставлять результаты по частям слов. Например, если оно задано по умолчанию в поле заголовки страницы под названием `Hello World!` оно будет обнаружено при поиске по части слова `Hel`.
 - **boost** (`int/float`) - Это позволяет вам устанавливать поля как более важные, чем другие. Установка этого значения на большое число в поле приведет к тому, что страницы со значениями в этом поле будут ранжироваться выше. В поле заголовка по умолчанию это значение равно 2 и 1 для всех остальных полей.
 - **es_extra** (`dict`) - Это поле позволяет разработчику устанавливать или переопределять любые настройки в поле сопоставлении для ElasticSearch. Используйте это, если вы хотите использовать любые функции ElasticSearch, которые еще не поддерживаются в Wagtail.

**`index.FilterField`**
Это добавлено в индекс поиска, но не используется для полнотекстового поиска. Вместо этого оно позволяет вам запускать фильтры в результатах поиска.

**`index.RelatedFields`**
Это позволяет вам индексировать поля из связанных объектов. Он работает во всех типах связанных полях, включая их обратные средства доступа.

Например, если у нас есть книга, которая имеет `ForeignKey` к ее автору, мы также можем вставить в книгу поля`name` и `date_of_birth` автора как в примере ниже:

>     class Book(models.Model, indexed.Indexed):
>         ...
>     
>         search_fields = [
>             index.SearchField('title'),
>             index.FilterField('published_date'),
>     
>             index.RelatedFields('author', [
>                 index.SearchField('name'),
>                 index.FilterField('date_of_birth'),
>             ]),
>         ]

Это позволит вам искать книги по имени их автора.

Это работает также и наоборот. Вы можете индексировать книги автора, позволяя искать по автору   названия книг, которые они опубликовали:

>     class Author(models.Model, indexed.Indexed):
>         ...
>     
>         search_fields = [
>             index.SearchField('name'),
>             index.FilterField('date_of_birth'),
>     
>             index.RelatedFields('books', [
>                 index.SearchField('title'),
>                 index.FilterField('published_date'),
>             ]),
>         ]

**Фильтрация по `index.RelatedFields`**

Фильтровать по `index.FilterFields` в `index.RelatedFields` используя `QuerySet` API невозможна. Однако поля индексируются, поэтому их можно использовать, запрашивая Elasticsearch вручную.

Фильтрация по `index.RelatedFields` используя `QuerySet` API планируется для будущего выпуска Wagtail.

**Индексирование вызовов и других атрибутов**

> **Примечание**
> Это не поддерживается в [**Database Backend**](http://docs.wagtail.io/en/v1.13.1/topics/search/backends.html#wagtailsearch-backends-database) (по умолчанию)

Поля поиска / фильтра не обязательно должны быть полями модели Django. Они также могут быть любым методом или атрибутом в вашей модели.

Одно из использований это индексирование по`get_*_display` которое Django создает автоматически для полей с функцией выбора.

>     from wagtail.wagtailsearch import index
>     
>     class EventPage(Page):
>         IS_PRIVATE_CHOICES = (
>             (False, "Public"),
>             (True, "Private"),
>         )
>     
>         is_private = models.BooleanField(choices=IS_PRIVATE_CHOICES)
>     
>         search_fields = Page.search_fields + [
>             # Index the human-readable string for searching.
>             index.SearchField('get_is_private_display'),
>     
>             # Index the boolean value for filtering.
>             index.FilterField('is_private'),
>         ]

Вызываемые объекты также предоставляют способ индексирования полей из связанных моделей. Как и в примере из [Inline Panels и Model Clusters](http://docs.wagtail.io/en/v1.13.1/reference/pages/panels.html#inline-panels), индексирование каждой BookPage  по названию через related_links:

>     class BookPage(Page):
>         # ...
>         def get_related_link_titles(self):
>             # Get list of titles and concatenate them
>             return '\n'.join(self.related_links.all().values_list('name', flat=True))
>     
>         search_fields = Page.search_fields + [
>             # ...
>             index.SearchField('get_related_link_titles'),
>         ]

**Индексирование пользовательских моделей**
---------------------------------------
Любая модель Django может быть проиндексирована и подключена к поиску.
Для этого наследуем `index.Indexed` и добавляем некоторые `search_fields` к модели.

>     from wagtail.wagtailsearch import index
>     
>     class Book(index.Indexed, models.Model):
>         title = models.CharField(max_length=255)
>         genre = models.CharField(max_length=255, choices=GENRE_CHOICES)
>         author = models.ForeignKey(Author)
>         published_date = models.DateTimeField()
>     
>         search_fields = [
>             index.SearchField('title', partial_match=True, boost=10),
>             index.SearchField('get_genre_display'),
>     
>             index.FilterField('genre'),
>             index.FilterField('author'),
>             index.FilterField('published_date'),
>         ]
>     
>     # As this model doesn't have a search method in its QuerySet, we have to call search directly on the backend
>     >>> from wagtail.wagtailsearch.backends import get_search_backend
>     >>> s = get_search_backend()
>     
>     # Run a search for a book by Roald Dahl
>     >>> roald_dahl = Author.objects.get(name="Roald Dahl")
>     >>> s.search("chocolate factory", Book.objects.filter(author=roald_dahl))
>     [<Book: Charlie and the chocolate factory>]
