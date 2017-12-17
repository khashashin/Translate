[Документация](http://docs.wagtail.io/en/v1.13.1/index.html) > [Руководство по использованию](http://docs.wagtail.io/en/v1.13.1/topics/index.html) > Формирование страницы используя StreamField/////////////////// [Редактировать на GitHub](https://github.com/wagtail/wagtail/blob/b0be9edf510ce414c4050195817094b9261bc945/docs/topics/permissions.rst)
___

Формирование страницы используя **StreamField**
----------
StreamField предоставляет модель редактирования контента, подходящую для страниц, которые не соответствуют фиксированной структуре - например, сообщения в блогах или новостные посты, где текст может быть помещен в подзаголовки, изображения, цитаты и видео. Он также подходит для более специализированных типов контента, таких как карты и диаграммы или для блога программирования, где нужно вставлять фрагменты кода. В этой модели эти различные типы контента представлены в виде последовательности «блоков», которые могут быть повторены и упорядочены в любом порядке. Для получения дополнительной информации о StreamField и о том, почему вы использовали бы его вместо «RichTextField» для статьи, см. Блог - [Rich text fields and faster horses](https://torchbox.com/blog/rich-text-fields-and-faster-horses/).

StreamField также предлагает богатый API для определения ваших собственных типов блоков, от простых коллекций подблоков (например, блок «person», состоящий из имени, фамилии и фотографии) до полностью настраиваемых компонентов с собственным интерфейсом редактирования. В базе данных содержимое StreamField хранится как JSON, гарантируя, что полный информационный контент поля сохраняется, а не только его представление в формате HTML.

Использование **StreamField**
-------------------------
`StreamField` - это поле модели, которое может быть определено в вашей модели страницы, как и любое другое поле:

>     from django.db import models
>     
>     from wagtail.wagtailcore.models import Page
>     from wagtail.wagtailcore.fields import StreamField
>     from wagtail.wagtailcore import blocks
>     from wagtail.wagtailadmin.edit_handlers import FieldPanel, StreamFieldPanel
>     from wagtail.wagtailimages.blocks import ImageChooserBlock
>     
>     class BlogPage(Page):
>         author = models.CharField(max_length=255)
>         date = models.DateField("Post date")
>         body = StreamField([
>             ('heading', blocks.CharBlock(classname="full title")),
>             ('paragraph', blocks.RichTextBlock()),
>             ('image', ImageChooserBlock()),
>         ])
>     
>         content_panels = Page.content_panels + [
>             FieldPanel('author'),
>             FieldPanel('date'),
>             StreamFieldPanel('body'),
>         ]

Примечание. StreamField не поддерживает обратную совместимость с другими типами полей, такими как RichTextField. Если вам нужно перенести существующее поле в StreamField, обратитесь к разделу [Миграция RichTextFields в StreamField](http://docs.wagtail.io/en/v1.13.1/topics/streamfield.html#streamfield-migrating-richtext).

Параметр `StreamField` - это список `(name, block_type)` кортежей. «name» используется для идентификации типа блока внутри шаблонов и внутреннего представления в JSON (и должны следовать стандартным правилам Python для имен переменных: нижний регистр и символы подчеркивания вместо пробелов) и «block_type» должен быть объектом определения блока, как описано ниже. (В качестве альтернативы `StreamField` может быть передан один экземпляр `StreamBlock` - см. [Структурные типы блоков](http://docs.wagtail.io/en/v1.13.1/topics/streamfield.html#structural-block-types).)

Это определяет набор доступных типов блоков, которые можно использовать в этом поле. Автор страницы может использовать эти блоки столько раз, сколько необходимо, в любом порядке.

`StreamField` также принимает необязательный аргумент `blank`, по умолчанию - false; применяя этот аргумент в состоянии по умолчанию т.е - false, должен быть предоставлен хотя бы один блок, чтобы поле считалось действительным.

Основные типы блоков
--------------------
Все типы блоков принимают следующие необязательные аргументы:

`default` - значение по умолчанию, которое должен получить новый «пустой» блок.
`label` - метка, отображаемая в интерфейсе редактора при обращении к этому блоку - по умолчанию используется префиксная версия имени блока (или в контексте, где имя не назначено, например, в `ListBlock` - пустая строка).
`icon` - имя значка, отображаемого для этого типа блока в меню выбора блока. Список имен значков см. В руководстве по стилю Wagtail, которое можно включить, добавив `wagtail.contrib.wagtailstyleguide` к вашему проекту в `INSTALLED_APPS`.
`template` - путь к шаблону Django, который будет использоваться для рендеринга этого блока на лицевой стороне. См. [Отображение шаблона](http://docs.wagtail.io/en/v1.13.1/topics/streamfield.html#template-rendering).
`group` - группа, используемая для категоризации текущего блока, то есть любые блоки с таким же именем группы будут показаны вместе в интерфейсе редактора с именем группы в качестве заголовка.

Основными типами блоков, предоставляемыми Wagtail, являются:

**CharBlock**
---------
`wagtail.wagtailcore.blocks.CharBlock`

Однострочный ввод текста. Принимаются следующие аргументы:

`required` **(по умолчанию: True)** - если true, поле не может быть пустым.
`max_length`, `min_length` - ограничивает максимальную или минимальную длину строки.
`help_text` - вспомогательный текст, будет отображаться рядом с полем.

**TextBlock**
---------
`wagtail.wagtailcore.blocks.TextBlock`

Многострочный ввод текста. Как и у `CharBlock` аргумент `required` (по умолчанию: True), так-же принимаются аргументы `max_length`, `min_length` и `help_text`.

**EmailBlock**
----------
`wagtail.wagtailcore.blocks.EmailBlock`

Однострочный ввод электронной почты, который следит, что-бы адрес электронной почты был написан верно т.е ввида user@domen.com. Аргумент `required` (по умолчанию: True), так-же принимается аргумент `help_text`.

Пример использования `EmailBlock`, см. [Example: PersonBlock](http://docs.wagtail.io/en/v1.13.1/topics/streamfield.html#streamfield-personblock-example)

**IntegerBlock**
------------
`wagtail.wagtailcore.blocks.IntegerBlock`

Однострочный целочисленный ввод, который проверяет, является ли целое число действительным целым числом. Аргумент `required` (по умолчанию: True), так-же принимаются аргументы `max_value`, `min_value` и `help_text`.

Для примера использования `IntegerBlock`, см. [Example: PersonBlock](http://docs.wagtail.io/en/v1.13.1/topics/streamfield.html#streamfield-personblock-example)

**FloatBlock**
----------
`wagtail.wagtailcore.blocks.FloatBlock`

Однострочный ввод числа с плавающей запятой, который подтверждает, что это значение является допустимым числом с плавающей запятой. Аргумент `required` (по умолчанию: True), так-же принимаются аргументы `max_value`, `min_value`.

**DecimalBlock**
------------
`wagtail.wagtailcore.blocks.DecimalBlock`

Однострочный десятичный ввод, который проверяет правильность десятичного числа. Аргумент `required` (по умолчанию: True), так-же принимаются аргументы `help_text`, `max_value`, `min_value`, `max_digits`  и  `decimal_places`.

Для примера использования `DecimalBlock`, см. [Example: PersonBlock](http://docs.wagtail.io/en/v1.13.1/topics/streamfield.html#streamfield-personblock-example)


**RegexBlock**
----------
`wagtail.wagtailcore.blocks.RegexBlock`

Однострочный ввод текста, который проверяет строку на ввод регулярного выражения. Регулярное выражение, используемое для проверки, должно быть предоставлено в качестве первого аргумента или в качестве аргумента под ключевым словом `regex`. Чтобы изменить текст сообщения ошибки, выводимое при проверки выражения, нужно передать в качестве словаря `error_messages` которое может сочетать в себе одно или оба ключевых слов как `required` (для сообщения, отображаемого при пустом значении) и `invalid` (для сообщения, отображаемого при несоответствии правильному значению):

>     blocks.RegexBlock(regex=r'^[0-9]{3}$', error_messages={
>         'invalid': "Not a valid library card number."
>     })

Принимаются аргументы `regex`, `help_text`, `required` (по умолчанию: True), `max_length`, `min_length` и `error_messages`

**URLBlock**
--------
`wagtail.wagtailcore.blocks.URLBlock`

Однострочный ввод текста, подтверждающий правильность URL. Принимаются аргументы `required` (по умолчанию: True), `max_length`, `min_length` и `help_text`

**BooleanBlock**
------------
`wagtail.wagtailcore.blocks.BooleanBlock`

Флажок. Принимаются аргументы `required` и `help_text`. Как и в случае с `BooleanField`  в Django, значение `required = True` (по умолчанию) указывает, что флажок должен быть отмечен галочкой для продолжения. Для флажка, который может быть отмечен или нет, вы должны явно передать значение `required = False`.

**DateBlock**
---------
`wagtail.wagtailcore.blocks.DateBlock`

Выбор даты. Принимаются аргументы `required` (по умолчанию: True), `help_text` и  `format`.

`format`**(default: None)** - формат даты. Это должен быть один из назначенных форматов, перечисленных в настройке [DATE_INPUT_FORMATS](https://docs.djangoproject.com/en/1.10/ref/settings/#std:setting-DATE_INPUT_FORMATS).  Если не указано, Wagtail будет использовать настройку `WAGTAIL_DATE_FORMAT` который возвращает формат типа: «% Y-% m-% d».

**TimeBlock**
---------
`wagtail.wagtailcore.blocks.TimeBlock`

Выбор времени. Принимаются аргументы `required` (по умолчанию: True)  и `help_text`.

**DateTimeBlock**
-------------
`wagtail.wagtailcore.blocks.DateTimeBlock`

Комбинированный выбор даты и времени. Принимаются аргументы `required` (по умолчанию: True), `help_text` и  `format`.

`format`**(default: None)** - формат даты. Это должен быть один из назначенных форматов, перечисленных в настройке [DATETIME_INPUT_FORMATS](https://docs.djangoproject.com/en/1.10/ref/settings/#std:setting-DATETIME_INPUT_FORMATS). Если не указано, Wagtail будет использовать настройку `WAGTAIL_DATETIME_FORMAT` который возвращает формат типа:  «%Y-%m-%d %H:%M».

**RichTextBlock**
-------------
`wagtail.wagtailcore.blocks.RichTextBlock`

Редактор WYSIWYG для создания форматированного текста, включая ссылки, полужирный, курсив и т. д. Принимается аргумент типа `features` , чтобы указать набор разрешенных функций (см. [Limiting features in a rich text field](http://docs.wagtail.io/en/v1.13.1/advanced_topics/customisation/page_editing_interface.html#rich-text-features)).

**RawHTMLBlock**
------------
`wagtail.wagtailcore.blocks.RawHTMLBlock`

Текстовая область для ввода необработанного HTML, который будет отображаться на странице без вывода. Это означает, что любые теги HTML, которые вы вводите в поле, будут отображаться как фактический HTML на странице вывода. поэтому, введя `<h1> hello world </ h1>` в поле, вы получите заголовок. Принимаются аргументы `required` (по умолчанию: True), `max_length`, `min_length` и `help_text`

> **Предупреждение**
> Когда этот блок используется, ничто не мешает редакторам вставлять
> вредоносные скрипты на страницу, включая сценарии, которые позволят
> редактору получать права администратора, когда другой администратор
> просматривает страницу. Не используйте этот блок, если полностью не
> доверяют вашим редакторам.

**BlockQuoteBlock**
---------------
`wagtail.wagtailcore.blocks.BlockQuoteBlock`

Текстовое поле, содержимое которого будет обернуто в пару тегов HTML `<blockquote>`. Принимаются аргументы `required` (по умолчанию: True), `max_length`, `min_length` и `help_text`

**ChoiceBlock**
-----------
`wagtail.wagtailcore.blocks.ChoiceBlock`

Выпадающий список с возможностью выбора из списка вариантов.  Принимаются следующие аргументы:

`choices` - Список вариантов, в любом формате поддерживаемом параметром [Django `choices`](https://docs.djangoproject.com/en/stable/ref/models/fields/#field-choices) для полей моделей, или возвращает вызываемый список.
`required` **(по умолчанию: True)** - если true, поле не может быть пустым.
`help_text` - вспомогательный текст, будет отображаться рядом с полем.

`ChoiceBlock` также может быть подклассифицирован для создания многоразового блока с тем же списком вариантов, которые будут используется везде. Например, определение блока, такое как:

>     blocks.ChoiceBlock(choices=[
>         ('tea', 'Tea'),
>         ('coffee', 'Coffee'),
>     ], icon='cup')

может быть переписана как подкласс `ChoiceBlock`:

>     class DrinksChoiceBlock(blocks.ChoiceBlock):
>         choices = [
>             ('tea', 'Tea'),
>             ('coffee', 'Coffee'),
>         ]
>     
>         class Meta:
>             icon = 'cup'

Определения `StreamField` могут затем ссылаться на `DrinksChoiceBlock()` вместо полного определения `ChoiceBlock`. Обратите внимание, что это работает только тогда, когда `choices` является фиксированным списком, а не вызываемым.

**PageChooserBlock**
----------------

`wagtail.wagtailcore.blocks.PageChooserBlock`

Элемент управления для выбора страницы, используя менеджер страниц Wagtail. Принимаются следующие аргументы:

`required` **(по умолчанию: True)** - если true, поле не может быть пустым.
`target_model` **(по умолчанию: Page)** - ограничить выбор одним или несколькими конкретными типами страниц. Принимает класс модели (Page), имя модели (в виде строки), список моделей или кортеж из них.
`can_choose_root` **(по умолчанию: False)** - Если true, редактор может выбрать основу структуры страниц. В основном это нежелательно, так как корень структуры страниц никогда не является полезной страницей, но в некоторых специализированных случаях это может быть уместно. Например, блок, обеспечивающий предоставление канала связанных статей, может использовать параметр `PageChooserBlock` чтобы выбрать, какой подраздел сайта будет доступен из корневого раздела.

**DocumentChooserBlock**
--------------------
`wagtail.wagtaildocs.blocks.DocumentChooserBlock`

Элемент управления, позволяющий редактору выбирать существующий документ или загружать новый. Принимается аргумент `required` (по умолчанию: True).

**ImageChooserBlock**
-----------------
`wagtail.wagtailimages.blocks.ImageChooserBlock`

Элемент управления, позволяющий редактору выбирать существующее изображение или загружать новый. Принимается аргумент `required` (по умолчанию: True).

**SnippetChooserBlock**
-----------------
`wagtail.wagtailsnippets.blocks.SnippetChooserBlock`

Элемент управления, позволяющий редактору выбирать отрывок (например фрагмент кода). Требуется один позиционный аргумент: the snippet class to choose from.  Принимается аргумент `required` (по умолчанию: True).

**EmbedBlock**
-----------------
`wagtail.wagtailembeds.blocks.EmbedBlock`

Поле, в котором редактор вводит URL-адрес элемента мультимедиа (например, видео YouTube), отображаемого в виде встроенных носителей на странице. Принимаются аргументы `required` (по умолчанию: True), `max_length`, `min_length` и `help_text`.

**StaticBlock**
-----------------
`wagtail.wagtailcore.blocks.StaticBlock`

Блок, который не имеет никаких полей, таким образом не передает никаких конкретных значений его шаблону во время рендеринга. Это может быть полезно, если вам нужно, чтобы редактор мог вставлять какой-то контент, который всегда один и тот же или не нужно настраивать в редакторе страниц. Например, адрес, код встраивания сторонних служб или более сложные фрагменты кода, если шаблон использует теги шаблонов. Некоторый текст по умолчанию (который содержит `label` аргумент, если вы его определите) будет отображаться в интерфейсе редактора, так что блок не выглядит пустым. Но вы также можете настроить его полностью, передав текстовую строку как аргумент`admin_text`:

>     blocks.StaticBlock(
>         admin_text='Latest posts: no configuration needed.',
>         # or admin_text=mark_safe('<b>Latest posts</b>: no configuration needed.'),
>         template='latest_posts.html')

StaticBlock также может быть подклассифицирован для создания многоразового блока с одинаковой конфигурацией везде, где он используется:

**Structural block types**
-----------------
В дополнение к основным типам блоков выше, можно определить новые типы блоков, состоящие из подблоков: например, блок «person», состоящий из подблоков для имени, фамилии и изображения или блок «carousel», состоящий из неограниченного количества блоков изображения. Такие структуры могут быть вложены в любую глубину, что позволяет иметь структуру содержащую список, или список структур.

**StructBlock**
-----------------
`wagtail.wagtailcore.blocks.StructBlock`

Блок, состоящий из фиксированной группы подблоков, которые будут отображаться вместе. В качестве первого аргумента принимает список `(name, block_definition)` кортежей:

>     ('person', blocks.StructBlock([
>         ('first_name', blocks.CharBlock()),
>         ('surname', blocks.CharBlock()),
>         ('photo', ImageChooserBlock(required=False)),
>         ('biography', blocks.RichTextBlock()),
>     ], icon='user'))

Еще список подблоков может быть предоставлен в подклассе StructBlock:

>     class PersonBlock(blocks.StructBlock):
>         first_name = blocks.CharBlock()
>         surname = blocks.CharBlock()
>         photo = ImageChooserBlock(required=False)
>         biography = blocks.RichTextBlock()
>     
>         class Meta:
>             icon = 'user'

Класс `Meta` поддерживает свойства `default`, `label`, `icon` и `template`  которые имеют те же значения, что и при определении их в конструкторе блока.

Это определяет `PersonBlock()` как тип блока, который можно повторно использовать столько раз, сколько вам понадобиться в определениях вашей модели:

>     body = StreamField([
>         ('heading', blocks.CharBlock(classname="full title")),
>         ('paragraph', blocks.RichTextBlock()),
>         ('image', ImageChooserBlock()),
>         ('person', PersonBlock()),
>     ])

Доступны дополнительные параметры для настройки отображения `StructBlock` в редакторе страниц - см. [Пользовательские интерфейсы редактирования для **StructBlock**](http://docs.wagtail.io/en/v1.13.1/topics/streamfield.html#custom-editing-interfaces-for-structblock).

**ListBlock**
---------
`wagtail.wagtailcore.blocks.ListBlock`

Блок, состоящий из множества подблоков, всех одинаковых типов. Редактор может добавлять неограниченное количество подблоков, а также переставлять и удалять их. Принимает определение подблока в качестве первого аргумента:    

>     ('ingredients_list', blocks.ListBlock(blocks.CharBlock(label="Ingredient")))

Любой тип блока действителен как тип подблока, включая структурные типы:

>     ('ingredients_list', blocks.ListBlock(blocks.StructBlock([
>         ('ingredient', blocks.CharBlock()),
>         ('amount', blocks.CharBlock(required=False)),
>     ])))

**StreamBlock**
---------
`wagtail.wagtailcore.blocks.StreamBlock`

Блок, состоящий из последовательности подблоков разных типов, которые могут быть смешаны и переупорядочены по желанию. Используется как общий механизм самого StreamField, но также может быть вложен или использоваться в других типах структурных блоков. В качестве первого аргумента принимает список `(name, block_definition)` кортежей:

>     ('carousel', blocks.StreamBlock(
>         [
>             ('image', ImageChooserBlock()),
>             ('quotation', blocks.StructBlock([
>                 ('text', blocks.TextBlock()),
>                 ('author', blocks.CharBlock()),
>             ])),
>             ('video', EmbedBlock()),
>         ],
>         icon='cogs'
>     ))

Как и в случае с StructBlock, список подблоков также может быть представлен как подкласс StreamBlock:

>     class CarouselBlock(blocks.StreamBlock):
>         image = ImageChooserBlock()
>         quotation = blocks.StructBlock([
>             ('text', blocks.TextBlock()),
>             ('author', blocks.CharBlock()),
>         ])
>         video = EmbedBlock()
>     
>         class Meta:
>             icon='cogs'

Поскольку `StreamField` принимает `StreamBlock` в качестве параметра, вместо списка типов блоков это позволяет повторно использовать общий набор типов блоков без их повторного определения:

>     class HomePage(Page):
>         carousel = StreamField(CarouselBlock(max_num=10, block_counts={'video': {'max_num': 2}}))

`StreamBlock` принимает следующие параметры как аргументы ключевого слова или свойства `Meta`:

`required` **(по умолчанию: True)** - если true, должен быть поставлен хотя бы один подблок. Это игнорируется при использовании StreamBlock в качестве блока верхнего уровня StreamField; в этом случае аргумент `blank` предпочтительнее.
`min_num` - минимальное количество подблоков, которые должен иметь поток.
`max_num` - максимальное количество подблоков, которые может иметь поток.
`block_counts` - Указывает минимальное и максимальное количество каждого типа блока, as a dictionary mapping block names to dicts with (optional) `min_num` and `max_num` fields.

**Example: `PersonBlock`**
---------
В этом примере показано, как описанные выше базовые типы блоков могут быть объединены в более сложный тип блока на основе StructBlock:

>     from wagtail.wagtailcore import blocks
>     
>     class PersonBlock(blocks.StructBlock):
>         name = blocks.CharBlock()
>         height = blocks.DecimalBlock()
>         age = blocks.IntegerBlock()
>         email = blocks.EmailBlock()
>     
>         class Meta:
>             template = 'blocks/person_block.html'

**Template rendering**
------------------
StreamField обеспечивает представление HTML для содержимого в целом, а также для каждого отдельного блока. Чтобы включить HTML-код на свою страницу, используйте тег `{%include_block%}`:

>     {% load wagtailcore_tags %}
>     
>      ...
>     
>     {% include_block page.body %}

В рендеринге по умолчанию каждый блок потока обернут в элемент `<div class = "block-my_block_name">` (где `my_block_name` - это имя блока, указанное в определении StreamField). Если вы хотите предоставить свою собственную разметку HTML, вы можете вместо этого создать цикл и перебирать значение поля и вызывать {% include_block %} на каждом блоке по очереди:

>     {% load wagtailcore_tags %}
>     
>      ...
>     
>     <article>
>         {% for block in page.body %}
>             <section>{% include_block block %}</section>
>         {% endfor %}
>     </article>

Для большего контроля над рендерингом конкретных типов блоков каждый объект блока предоставляет свойства `block_type` и `value`:

>     {% load wagtailcore_tags %}
>     
>      ...
>     
>     <article>
>         {% for block in page.body %}
>             {% if block.block_type == 'heading' %}
>                 <h1>{{ block.value }}</h1>
>             {% else %}
>                 <section class="block-{{ block.block_type }}">
>                     {% include_block block %}
>                 </section>
>             {% endif %}
>         {% endfor %}
>     </article>

По умолчанию каждый блок визуализируется с использованием простой, минимальной разметки HTML или вообще без разметки. Например, значение CharBlock отображается как обычный текст, а ListBlock выводит дочерние блоки в обертке `<ul>`. Чтобы переопределить это с помощью собственного пользовательского HTML-рендеринга, вы можете передать аргумент «template» блоку, указав имя файла шаблона для визуализации. Это особенно полезно для пользовательских типов блоков, полученных из StructBlock:

>     ('person', blocks.StructBlock(
>         [
>             ('first_name', blocks.CharBlock()),
>             ('surname', blocks.CharBlock()),
>             ('photo', ImageChooserBlock(required=False)),
>             ('biography', blocks.RichTextBlock()),
>         ],
>         template='myapp/blocks/person.html',
>         icon='user'
>     ))

Или, если он определен как подкласс StructBlock:

>     class PersonBlock(blocks.StructBlock):
>         first_name = blocks.CharBlock()
>         surname = blocks.CharBlock()
>         photo = ImageChooserBlock(required=False)
>         biography = blocks.RichTextBlock()
>     
>         class Meta:
>             template = 'myapp/blocks/person.html'
>             icon = 'user'

Внутри шаблона значение блока доступно как переменная «value»:

>     {% load wagtailimages_tags %}
>     
>     <div class="person">
>         {% image value.photo width-400 %}
>         <h2>{{ value.first_name }} {{ value.surname }}</h2>
>         {{ value.biography }}
>     </div>

Поскольку `first_name`, `surname`, `photo` и `biography` определяются как блоки сами по себе, это также может быть записано как:

>     {% load wagtailcore_tags wagtailimages_tags %}
>     
>     <div class="person">
>         {% image value.photo width-400 %}
>         <h2>{% include_block value.first_name %} {% include_block value.surname %}</h2>
>         {% include_block value.biography %}
>     </div>

Написание `{{my_block}}` примерно эквивалентно `{% include_block my_block%}`, но краткая форма более ограничена, поскольку она не передает переменные из вызывающего шаблона, такие как «request» или «page»; по этой причине рекомендуется использовать его только для простых значений, которые не отображают собственный HTML. Например, если наш `PersonBlock` использовал бы шаблон:

>     {% load wagtailimages_tags %}
>     
>     <div class="person">
>         {% image value.photo width-400 %}
>         <h2>{{ value.first_name }} {{ value.surname }}</h2>
>     
>         {% if request.user.is_authenticated %}
>             <a href="#">Contact this person</a>
>         {% endif %}
>     
>         {{ value.biography }}
>     </div>

то проверка `request.user.is_authenticated` не будет корректно работать при рендеринге блока через тег `{{...}}`:

>     {# Incorrect: #}
>     
>     {% for block in page.body %}
>         {% if block.block_type == 'person' %}
>             <div>
>                 {{ block }}
>             </div>
>         {% endif %}
>     {% endfor %}
>     
>     {# Correct: #}
>     
>     {% for block in page.body %}
>         {% if block.block_type == 'person' %}
>             <div>
>                 {% include_block block %}
>             </div>
>         {% endif %}
>     {% endfor %}

Как и тэг `{% include%}` в Django, `{% include_block%}` в Wagtail, также позволяет передавать дополнительные переменные включенному шаблону,  через синтаксис
`{% include_block my_block with foo = "bar"%}`:

>     {# In page template: #}
>     
>     {% for block in page.body %}
>         {% if block.block_type == 'person' %}
>             {% include_block block with classname="important" %}
>         {% endif %}
>     {% endfor %}
>     
>     {# In PersonBlock template: #}
>     
>     <div class="{{ classname }}">
>         ...
>     </div>

Также поддерживается синтаксис типа `{% include_block my_block с foo = "bar" only%}`, чтобы указать, что никакие переменные из родительского шаблона, отличного от foo, не будут переданы дочернему шаблону.

Помимо передачи переменных из родительского шаблона, блочные подклассы могут передавать собственные дополнительные шаблоны, переопределяя метод `get_context()`:

>     import datetime
>     
>     class EventBlock(blocks.StructBlock):
>         title = blocks.CharBlock()
>         date = blocks.DateBlock()
>     
>         def get_context(self, value, parent_context=None):
>             context = super(EventBlock, self).get_context(value, parent_context=parent_context)
>             context['is_happening_today'] = (value['date'] == datetime.date.today())
>             return context
>     
>         class Meta:
>             template = 'myapp/blocks/event.html'

В этом примере переменная `is_happening_today` будет доступна в шаблоне блока. Аргумент `parent_context` доступен, когда блок визуализируется с помощью тега `{% include_block%}` и является типом переменных, переданных из вызывающего шаблона.

**BoundBlocks and values**
----------------------
Все типы блоков, а не только `StructBlock`, принимают параметр `template`, чтобы определить, как они будут отображаться на странице. Однако для блоков, которые обрабатывают базовые типы данных Python, такие как `CharBlock` и `IntegerBlock`, существуют некоторые ограничения на то, где шаблон будет действовать, поскольку эти встроенные типы (`str`, `int` и т. д.) не могут не могут быть обработаны при рендеринге в шаблон. В качестве примера рассмотрим следующее определение блока:

>     class HeadingBlock(blocks.CharBlock):
>         class Meta:
>             template = 'blocks/heading.html'

где `blocks/heading.html` состоит из:

>    `<h1>{{ value }}</h1>`

Это дает нам блок, который ведет себя как обычное текстовое поле, но переносит его вывод в тегах `<h1>` всякий раз, когда он отображается:

>     class BlogPage(Page):
>         body = StreamField([
>             # ...
>             ('heading', HeadingBlock()),
>             # ...
>         ])
>

>     {% load wagtailcore_tags %}
>     
>     {% for block in page.body %}
>         {% if block.block_type == 'heading' %}
>             {% include_block block %}  {# This block will output its own <h1>...</h1> tags. #}
>         {% endif %}
>     {% endfor %}

Такая схема - значение, которое предположительно представляет собой обычную текстовую строку, но имеет собственное пользовательское представление HTML при рендеринге - обычно было бы очень грязной задачей для достижения в Python, но здесь он работает, потому что элементы, которые вы получаете при итерации по StreamField, на самом деле не являются «родными» значениями этих блоков. Вместо этого каждый элемент возвращается как экземпляр `BoundBlock` - объект, представляющий сопряжение значения и его определение блока. Следя за определением блока, `BoundBlock` всегда знает, какой шаблон отображать. Чтобы получить базовое значение - в данном примере, текстовое содержимое заголовка - вам нужно получить доступ к `block.value`. И в самом деле, если вы хотите вывести
`{% include_block block.value%}` на странице, вы увидите, что он отображается как обычный текст без тегов `<h1>`.

(Точнее, элементы, возвращаемые при переходе по `StreamField`, являются экземплярами класса `StreamChild`, который предоставляет свойство `block_type`, а также `value`.)

Опытные разработчики Django могут счесть это полезным по сравнению с классом `BoundField` в структуре форм Django, который представляет собой спаривание значения поля формы с его соответствующим определением, и поэтому знает, как рендерить значение как поле формы HTML.

В большинстве случаев вам не нужно беспокоиться об этих внутренних деталях; Wagtail будет использовать рендеринг шаблонов, где бы вы ни ожидали. Однако есть определенные случаи, когда это не совсем так, а именно, при доступе к дочерним элементам ListBlock или StructBlock. В этих случаях нет обертки `BoundBlock`, and so the item cannot be relied upon to know its own template rendering. Например, рассмотрите следующую настройку, где наш `HeadingBlock` является дочерним элементом `StructBlock`:

>     class EventBlock(blocks.StructBlock):
>         heading = HeadingBlock()
>         description = blocks.TextBlock()
>         # ...
>     
>         class Meta:
>             template = 'blocks/event.html'

В `blocks/event.html`:

>     {% load wagtailcore_tags %}
>     
>     <div class="event {% if value.heading == 'Party!' %}lots-of-balloons{% endif %}">
>         {% include_block value.heading %}
>         - {% include_block value.description %}
>     </div>

В этом случае `value.heading` возвращает простое строковое значение, а не `BoundBlock`; это необходимо, потому что иначе сравнение в `{% if value.heading == 'Party!' %}` никогда не будет успешным. Это, в свою очередь, означает, что `{% include_block value.heading%}` отображается как простая строка без тегов `<h1>`. Чтобы получить рендеринг HTML, вам нужно явно получить доступ к экземпляру `BoundBlock` через `value.bound_blocks.heading`:

>     {% load wagtailcore_tags %}
>     
>     <div class="event {% if value.heading == 'Party!' %}lots-of-balloons{% endif %}">
>         {% include_block value.bound_blocks.heading %}
>         - {% include_block value.description %}
>     </div>

На практике, вероятно, было бы более естественным и читабельным сделать ярлык `<h1>` явным в шаблоне `EventBlock`:

>     {% load wagtailcore_tags %}
>     
>     <div class="event {% if value.heading == 'Party!' %}lots-of-balloons{% endif %}">
>         <h1>{{ value.heading }}</h1>
>         - {% include_block value.description %}
>     </div>

Это ограничение не относится к значениям `StructBlock` и `StreamBlock` в качестве дочерних элементов `StructBlock`, поскольку Wagtail реализует их как сложные объекты, которые знают свой собственный рендеринг шаблон, даже если они не завернуты в `BoundBlock`. Например, если `StructBlock` вложен в другой `StructBlock`, как тут:

>     class EventBlock(blocks.StructBlock):
>         heading = HeadingBlock()
>         description = blocks.TextBlock()
>         guest_speaker = blocks.StructBlock([
>             ('first_name', blocks.CharBlock()),
>             ('surname', blocks.CharBlock()),
>             ('photo', ImageChooserBlock()),
>         ], template='blocks/speaker.html')

то `{% include_block value.guest_speaker%}` в шаблоне `EventBlock` запускает рендеринг шаблона из `blocks/speaker.html`, как предполагалось.

Таким образом, взаимодействия между `BoundBlocks` и равными значениями работают в соответствии со следующими правилами:

 1. При повторении значения `StreamField` или `StreamBlock` (как в {% for block in page.body%}) вы вернете последовательность BoundBlocks.
 2. Если у вас есть экземпляр BoundBlock, вы можете получить доступ к простому значению как `block.value`.
 3. Доступ к дочернему элементу StructBlock (как в `value.heading`) приведет к возврату простого значения; для получения BoundBlock вместо этого используйте `value.bound_blocks.heading`.
 4. Значение ListBlock - это простой список Python; итерация по ней возвращает простые дочерние значения.
 5. Значения StructBlock и StreamBlock всегда знают, как отображать свои собственные шаблоны, даже если у вас есть простое значение, а не BoundBlock.

**Дополнительные способы редактирования для `StructBlock`**
-----------------------------------------
Чтобы настроить стиль `StructBlock`, как он отображается в редакторе страниц, вы можете указать атрибут `form_classname` (либо как аргумент ключевого слова конструктору `StructBlock`, или в `Meta` подклассе) для переопределения значения `struct-block` по умолчанию:

>     class PersonBlock(blocks.StructBlock):
>         first_name = blocks.CharBlock()
>         surname = blocks.CharBlock()
>         photo = ImageChooserBlock(required=False)
>         biography = blocks.RichTextBlock()
>     
>         class Meta:
>             icon = 'user'
>             form_classname = 'person-block struct-block'

Затем вы можете предоставить настраиваемый CSS для этого блока, ориентированный на указанное имя класса, с помощью `insert_editor_css`

> **Примечание**
>
>Wagtail имеет некоторые встроенные стили для класса `struct-block` и других связанных элементов. Если вы укажете значение
> `form_classname`, оно перезапишет классы, которые уже применяются к

Для более обширных настроек, требующих изменений в разметке HTML, вы можете переопределить атрибут `form_template` в `Meta`, чтобы указать свой собственный путь к шаблону. В этом шаблоне доступны следующие переменные:

`children`  - `OrderedDict` из `BoundBlock` блоков для всех дочерних блоков, составляющих этот `StructBlock`; ваш шаблон будет вызывать `render_form` для каждого из них.
`help_text` - вспомогательный текст, будет отображаться рядом с полем.
`classname` - имя класса передается как `form_classname` (по умолчанию в - `struct-block`).
`block_definition` - экземпляр `StructBlock`, который определяет этот блок.
`prefix` - префикс, используемый для полей формы для этого экземпляра блока, уникален по всей форме.

Чтобы добавить дополнительные переменные, вы можете переопределить метод блока `get_form_context`:

>     class PersonBlock(blocks.StructBlock):
>         first_name = blocks.CharBlock()
>         surname = blocks.CharBlock()
>         photo = ImageChooserBlock(required=False)
>         biography = blocks.RichTextBlock()
>     
>         def get_form_context(self, value, prefix='', errors=None):
>             context = super(PersonBlock, self).get_form_context(value, prefix=prefix, errors=errors)
>             context['suggested_first_names'] = ['John', 'Paul', 'George', 'Ringo']
>             return context
>     
>         class Meta:
>             icon = 'user'
>             form_template = 'myapp/block_forms/person.html'

**Пользовательские типы блоков**
----------------------------
Если вам нужно реализовать пользовательский интерфейс или обрабатывать тип данных, который не предоставляется встроенными типами блоков Wagtail (и не может быть построена на структуре существующих полей), вы можете определить свои собственные типы блоков. Для получения дополнительной информации обратитесь к исходному коду встроенных классов блоков Wagtail.

Для типов блоков, которые просто переносят существующие поля форм Django, Wagtail предоставляет абстрактный класс `wagtail.wagtailcore.blocks.FieldBlock`. Подклассам просто нужно установить свойство поля, которое возвращает объект поля формы:

>     class IPAddressBlock(FieldBlock):
>         def __init__(self, required=True, help_text=None, **kwargs):
>             self.field = forms.GenericIPAddressField(required=required, help_text=help_text)
>             super(IPAddressBlock, self).__init__(**kwargs)

**Миграции**
========

**Определения StreamField в миграциях**
-----------------------------------------
Как и в любом поле модели в Django, любые изменения в определении модели, влияющие на StreamField, приведут к созданию файла миграции, который содержит «замороженную» копию этого определения поля. Поскольку определение StreamField более сложное, чем типичное поле модели, существует повышенная вероятность того, что определения из вашего проекта будут импортированы в миграцию, что впоследствии вызовет проблемы, если эти определения будут перемещены или удалены.

Чтобы это смягчить, StructBlock, StreamBlock и ChoiceBlock реализуют дополнительную логику для обеспечения того, чтобы любые подклассы этих блоков были деконструированы на простые экземпляры StructBlock, StreamBlock и ChoiceBlock, таким образом, миграции не содержат ссылок на ваши пользовательские определения классов. Это возможно, потому что эти типы блоков предоставляют стандартный шаблон для наследования и знают, как восстановить определение блока для любого подкласса, который следует за этим шаблоном.

Если вы подклассифицируете любой другой класс блока, например `FieldBlock`, вам нужно будет сохранить это определение класса на протяжении всего времени в проекте, или реализовать собственный [метод деконструирования](https://docs.djangoproject.com/en/1.9/topics/migrations/#custom-deconstruct-method) который выражает ваш блок полностью с точки зрения классов которые должны остаться на месте. Аналогичным образом, если вы настраиваете подкласс StructBlock, StreamBlock или ChoiceBlock в том месте, где он больше не может быть выражен как экземпляр базового типа блока - например, если вы добавите дополнительные аргументы в конструктор - вам нужно будет предоставить свой собственный `deconstruct` метод.

**Миграция RichTextFields в StreamField**
-------------------------------------
Если вы измените существующий RichTextField на StreamField и сделаете миграцию, перенос будет завершен без ошибок, поскольку оба поля используют текстовый столбец в базе данных. Однако StreamField использует представление JSON для своих данных, поэтому существующий текст необходимо преобразовать с миграцией данных, чтобы он снова стал доступным. Чтобы это сработало, StreamField должен включать RichTextBlock в качестве одного из доступных типов блоков. Затем поле можно преобразовать, сделав новую миграцию (`./manage.py makemigrations --empty myapp`) и редактируя его следующим образом (в этом примере поле «body» модели `demo.BlogPage` преобразуется в StreamField с RichTextBlock с именем `rich_text`):

>     # -*- coding: utf-8 -*-
>     from __future__ import unicode_literals
>     
>     from django.db import models, migrations
>     from wagtail.wagtailcore.rich_text import RichText
>     
>     
>     def convert_to_streamfield(apps, schema_editor):
>         BlogPage = apps.get_model("demo", "BlogPage")
>         for page in BlogPage.objects.all():
>             if page.body.raw_text and not page.body:
>                 page.body = [('rich_text', RichText(page.body.raw_text))]
>                 page.save()
>     
>     
>     def convert_to_richtext(apps, schema_editor):
>         BlogPage = apps.get_model("demo", "BlogPage")
>         for page in BlogPage.objects.all():
>             if page.body.raw_text is None:
>                 raw_text = ''.join([
>                     child.value.source for child in page.body
>                     if child.block_type == 'rich_text'
>                 ])
>                 page.body = raw_text
>                 page.save()
>     
>     
>     class Migration(migrations.Migration):
>     
>         dependencies = [
>             # leave the dependency line from the generated migration intact!
>             ('demo', '0001_initial'),
>         ]
>     
>         operations = [
>             migrations.RunPython(
>                 convert_to_streamfield,
>                 convert_to_richtext,
>             ),
>         ]

Обратите внимание, что вышеуказанная миграция будет работать только с опубликованными страницами. Если вам также нужно перенести черновики страниц и изменения страниц, тогда отредактируйте новую миграцию данных, как показано в следующем примере:

>     # -*- coding: utf-8 -*-
>     from __future__ import unicode_literals
>     
>     import json
>     
>     from django.core.serializers.json import DjangoJSONEncoder
>     from django.db import migrations, models
>     
>     from wagtail.wagtailcore.rich_text import RichText
>     
>     
>     def page_to_streamfield(page):
>         changed = False
>         if page.body.raw_text and not page.body:
>             page.body = [('rich_text', {'rich_text': RichText(page.body.raw_text)})]
>             changed = True
>         return page, changed
>     
>     
>     def pagerevision_to_streamfield(revision_data):
>         changed = False
>         body = revision_data.get('body')
>         if body:
>             try:
>                 json.loads(body)
>             except ValueError:
>                 revision_data['body'] = json.dumps(
>                     [{
>                         "value": {"rich_text": body},
>                         "type": "rich_text"
>                     }],
>                     cls=DjangoJSONEncoder)
>                 changed = True
>             else:
>                 # It's already valid JSON. Leave it.
>                 pass
>         return revision_data, changed
>     
>     
>     def page_to_richtext(page):
>         changed = False
>         if page.body.raw_text is None:
>             raw_text = ''.join([
>                 child.value['rich_text'].source for child in page.body
>                 if child.block_type == 'rich_text'
>             ])
>             page.body = raw_text
>             changed = True
>         return page, changed
>     
>     
>     def pagerevision_to_richtext(revision_data):
>         changed = False
>         body = revision_data.get('body', 'definitely non-JSON string')
>         if body:
>             try:
>                 body_data = json.loads(body)
>             except ValueError:
>                 # It's not apparently a StreamField. Leave it.
>                 pass
>             else:
>                 raw_text = ''.join([
>                     child['value']['rich_text'] for child in body_data
>                     if child['type'] == 'rich_text'
>                 ])
>                 revision_data['body'] = raw_text
>                 changed = True
>         return revision_data, changed
>     
>     
>     def convert(apps, schema_editor, page_converter, pagerevision_converter):
>         BlogPage = apps.get_model("demo", "BlogPage")
>         for page in BlogPage.objects.all():
>     
>             page, changed = page_converter(page)
>             if changed:
>                 page.save()
>     
>             for revision in page.revisions.all():
>                 revision_data = json.loads(revision.content_json)
>                 revision_data, changed = pagerevision_converter(revision_data)
>                 if changed:
>                     revision.content_json = json.dumps(revision_data, cls=DjangoJSONEncoder)
>                     revision.save()
>     
>     
>     def convert_to_streamfield(apps, schema_editor):
>         return convert(apps, schema_editor, page_to_streamfield, pagerevision_to_streamfield)
>     
>     
>     def convert_to_richtext(apps, schema_editor):
>         return convert(apps, schema_editor, page_to_richtext, pagerevision_to_richtext)
>     
>     
>     class Migration(migrations.Migration):
>     
>         dependencies = [
>             # leave the dependency line from the generated migration intact!
>             ('demo', '0001_initial'),
>         ]
>     
>         operations = [
>             migrations.RunPython(
>                 convert_to_streamfield,
>                 convert_to_richtext,
>             ),
>         ]
