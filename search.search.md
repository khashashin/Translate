[Документация](http://docs.wagtail.io/en/v1.13.1/index.html) > [Руководство по использованию](http://docs.wagtail.io/en/v1.13.1/topics/index.html) > [Поиск](http://docs.wagtail.io/en/v1.13.1/topics/search/index.html) > Поиск ///////// [Редактировать на GitHub](https://github.com/wagtail/wagtail/blob/b0be9edf510ce414c4050195817094b9261bc945/docs/topics/permissions.rst)
___

**Поиск**
--------------

**Поиск QuerySets**

Поиск Wagtail построен на [QuerySet API](https://docs.djangoproject.com/en/1.8/ref/models/querysets/) Django. Вы должны иметь возможность искать любой Django QuerySet при условии, что модель и поля, которые были отфильтрованы, были добавлены в индекс поиска.

**Поиск страниц**

Wagtail предоставляет ярлык для поиска страниц: `QuerySet` `search()` метод. Вы можете вызвать его на любом `PageQuerySet`. Например:

>     # Search future EventPages
>     >>> from wagtail.wagtailcore.models import EventPage
>     >>> EventPage.objects.filter(date__gt=timezone.now()).search("Hello world!")

Все другие методы `PageQuerySet`  могут использоваться с `search()`. Например:

>     # Search all live EventPages that are under the events index
>     >>> EventPage.objects.live().descendant_of(events_index).search("Event")
>     [<EventPage: Event 1>, <EventPage: Event 2>]
>

> **Примечание** Метод `search()` будет конвертировать ваш `QuerySet` в экземпляр одного из Wagtail `SearchResults` класс (в зависимости от
> бэкэнда).  Это означает, что вы должны выполнить фильтрацию перед
> вызовом `search()`.

**Поиск изображений, документов и пользовательских моделей**
--------------------------------------------------------
Модели документы и изображения в Wagtail поддерживают метод `search`  по своим `QuerySet`, также как и страницы:

>     >>> from wagtail.wagtailimages.models import Image>     
>     >>> Image.objects.filter(uploaded_by_user=user).search("Hello")
>     [<Image: Hello>, <Image: Hello world!>]

Пользовательские модели можно найти, используя метод  `search` по поиску напрямую в бэкэнд:

>     >>> from myapp.models import Book
>     >>> from wagtail.wagtailsearch.backends import get_search_backend
>     
>     # Search books
>     >>> s = get_search_backend()
>     >>> s.search("Great", Book)
>     [<Book: Great Expectations>, <Book: The Great Gatsby>]

Вы также можете передать QuerySet в метод `search`, который позволяет добавлять фильтры в результаты поиска:

>     >>> from myapp.models import Book
>     >>> from wagtail.wagtailsearch.backends import get_search_backend
>     
>     # Search books
>     >>> s = get_search_backend()
>     >>> s.search("Great", Book.objects.filter(published_date__year__lt=1900))
>     [<Book: Great Expectations>]

**Указание полей для поиска**
-------------------------
По умолчанию Wagtail будет искать все поля, которые были проиндексированы, используя `index.SearchField`.

Это может быть ограничено к определенному набору полей используя `fields`  аргумент:

>     # Search just the title field
>     >>> EventPage.objects.search("Event", fields=["title"])
>     [<EventPage: Event 1>, <EventPage: Event 2>]

**Изменение поведения поиска**
--------------------------
**Оператор поиска**

Оператор поиска указывает, как должен вести себя поиск, когда пользователь вводит несколько терминов поиска. Возможны два возможных значения:

 - “or” - Результаты должны совпадать как минимум с одним термином (по умолчанию для Elasticsearch)
 - “and” - Результаты должны соответствовать всем условиям (по умолчанию для поиска в базе данных)

Оба оператора имеют преимущества и недостатки. “or” оператор вернет много других результатов, но, скорее всего, будет содержать множество результатов, которые не являются релевантными. “and” оператор возвращает результаты, содержащие все условия поиска, но требует от пользователя уточнения их запросов.

Мы рекомендуем использовать “or” оператор для возвращения ответа по релевантности и “and” оператор для возвращения ответа по чему-либо еще (примечание: бэкэнд базы данных в настоящее время не поддерживает оператор “or”).

Вот пример использования аргумента `operator`:

>     # The database contains a "Thing" model with the following items:
>     # - Hello world
>     # - Hello
>     # - World
>     
>     
>     # Search with the "or" operator
>     >>> s = get_search_backend()
>     >>> s.search("Hello world", Things, operator="or")
>     
>     # All records returned as they all contain either "hello" or "world"
>     [<Thing: Hello World>, <Thing: Hello>, <Thing: World>]
>     
>     
>     # Search with the "and" operator
>     >>> s = get_search_backend()
>     >>> s.search("Hello world", Things, operator="and")
>     
>     # Only "hello world" returned as that's the only item that contains both terms
>     [<Thing: Hello world>]

Для моделей страниц, изображений и документов, аргумент `operator`  также поддерживается в методе QuerySet `search`:

>     >>> Page.objects.search("Hello world", operator="or")
>     
>     # All pages containing either "hello" or "world" are returned
>     [<Page: Hello World>, <Page: Hello>, <Page: World>]

**Пользовательское упорядочивание**
По умолчанию результаты поиска упорядочены по релевантности, если бэкэнд поддерживает это. Чтобы сохранить существующее упорядочивание QuerySet, аргумент `order_by_relevance` необходимо установить  `False` в методе `search()`. Например:

>     # Get a list of events ordered by date
>     >>> EventPage.objects.order_by('date').search("Event", order_by_relevance=False)
>     
>     # Events ordered by date
>     [<EventPage: Easter>, <EventPage: Halloween>, <EventPage: Christmas>]

**Присваивание результату поиска «оценки»**

Для каждого согласованного результата Elasticsearch подсчитывает «оценку», который представляет собой число показывающее насколько релевантен результат, основанный на запросе пользователя. Результаты обычно упорядочиваются на основе оценки.

Есть случаи, когда доступ к счету полезен (например, программное объединение двух запросов для разных моделей). Вы можете добавить оценку к каждому результату, вызвав метод `.annotate_score(field)` в `SearchQuerySet`. Например:

>     >>> events = EventPage.objects.search("Event").annotate_score("_score")
>     >>> for event in events:
>     ...    print(event.title, event._score)
>     ...
>     ("Easter", 2.5),
>     ("Haloween", 1.7),
>     ("Christmas", 1.5),

Обратите внимание, что сам счет произволен, и он полезен только для сравнения результатов для одного и того же запроса.

**Пример страницы поиска**

Вот пример Django, который можно использовать для добавления страницы поиска на ваш сайт:

>     # views.py
>     
>     from django.shortcuts import render
>     
>     from wagtail.wagtailcore.models import Page
>     from wagtail.wagtailsearch.models import Query
>     
>     
>     def search(request):
>         # Search
>         search_query = request.GET.get('query', None)
>         if search_query:
>             search_results = Page.objects.live().search(search_query)
>     
>             # Log the query so Wagtail can suggest promoted results
>             Query.get(search_query).add_hit()
>         else:
>             search_results = Page.objects.none()
>     
>         # Render template
>         return render(request, 'search_results.html', {
>             'search_query': search_query,
>             'search_results': search_results,
>         })

И вот шаблон для этого:

>     {% extends "base.html" %}
>     {% load wagtailcore_tags %}
>     
>     {% block title %}Search{% endblock %}
>     
>     {% block content %}
>         <form action="{% url 'search' %}" method="get">
>             <input type="text" name="query" value="{{ search_query }}">
>             <input type="submit" value="Search">
>         </form>
>     
>         {% if search_results %}
>             <ul>
>                 {% for result in search_results %}
>                     <li>
>                         <h4><a href="{% pageurl result %}">{{ result }}</a></h4>
>                         {% if result.search_description %}
>                             {{ result.search_description|safe }}
>                         {% endif %}
>                     </li>
>                 {% endfor %}
>             </ul>
>         {% elif search_query %}
>             No results found
>         {% else %}
>             Please type something into the search box
>         {% endif %}
>     {% endblock %}

**Продвинутые результаты поиска**

«Продвигаемые результаты поиска» позволяют редакторам напрямую связывать релевантный контент с поисковыми запросами, поэтому страницы результатов могут содержать кураторский контент в дополнение к результатам поисковой системы.

Эта функциональность обеспечивается модулем `wagtailsearchpromotionscontrib`.
