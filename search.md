[Документация](http://docs.wagtail.io/en/v1.13.1/index.html) > [Руководство по использованию](http://docs.wagtail.io/en/v1.13.1/topics/index.html) > Поиск /////////////////////////// [Редактировать на GitHub](https://github.com/wagtail/wagtail/blob/b0be9edf510ce414c4050195817094b9261bc945/docs/topics/permissions.rst)
___
Wagtail обеспечивает всесторонний и расширяемый интерфейс поиска. Кроме того, он предоставляет способы поиска на выбор редактора. Wagtail также собирает простую статистику по запросам, выполненным через интерфейс поиска.

  - [Индексирование](http://docs.wagtail.io/en/v1.13.1/topics/search/indexing.html)
	  - [Обновление индекса](http://docs.wagtail.io/en/v1.13.1/topics/search/indexing.html#updating-the-index)
	  - [Индексирование дополнительных полей](http://docs.wagtail.io/en/v1.13.1/topics/search/indexing.html#indexing-extra-fields)
	  - [Индексирование пользовательских моделей](http://docs.wagtail.io/en/v1.13.1/topics/search/indexing.html#indexing-custom-models)
  - [Поиск](http://docs.wagtail.io/en/v1.13.1/topics/search/searching.html)
	  - [Поиск запросов](http://docs.wagtail.io/en/v1.13.1/topics/search/searching.html#searching-querysets)
	  - [Пред просмотр результатов поиска](http://docs.wagtail.io/en/v1.13.1/topics/search/searching.html#an-example-page-search-view)
	  - [Расширенные результаты поиска](http://docs.wagtail.io/en/v1.13.1/topics/search/searching.html#promoted-search-results)
  - [Надстройки](http://docs.wagtail.io/en/v1.13.1/topics/search/backends.html)
	  - [`AUTO_UPDATE`](http://docs.wagtail.io/en/v1.13.1/topics/search/backends.html#auto-update)
	  - [`ATOMIC_REBUILD`](http://docs.wagtail.io/en/v1.13.1/topics/search/backends.html#atomic-rebuild)
	  - [`BACKEND`](http://docs.wagtail.io/en/v1.13.1/topics/search/backends.html#backend)

**Индексирование**
--------------
Чтобы сделать объекты доступными для поиска, они должны быть сначала добавлены в индекс поиска. Это включает в себя настройку моделей и полей, которые вы хотите индексировать (поиск по страницам, изображениям и документам, Wagtail поддерживает из коробки), а затем фактически включение их в индекс. См. [Обновление индекса](http://docs.wagtail.io/en/v1.13.1/topics/search/indexing.html#wagtailsearch-indexing-update) - для дополнительной информации о том, как сохранить объекты в индексе поиска в синхронизации с объектами в вашей базе данных.

Если вы создали дополнительные поля в подклассе `Page` или `Image`, где вы хотели бы чтобы работал поиск, см. [индексирование пользовательских моделей](http://docs.wagtail.io/en/v1.13.1/topics/search/indexing.html#wagtailsearch-indexing-models).

**Поиск**
-----
Wagtail предоставляет API для выполнения поисковых запросов в ваших моделях. Вы также можете выполнять поисковые запросы по Django QuerySets.

См. [Поиск](http://docs.wagtail.io/en/v1.13.1/topics/search/searching.html#wagtailsearch-searching).

**Backends**
--------
Wagtail предусматривает три бэкэнда для хранения индекса поиска и выполнение поисковых запросов: Elasticsearch, база данных и PostgreSQL (Требуется Django> = 1.10). Можно также использовать свою собственную поисковую базу.

См. [Backends](http://docs.wagtail.io/en/v1.13.1/topics/search/backends.html#wagtailsearch-backends)
