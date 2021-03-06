[Документация](http://docs.wagtail.io/en/v1.13.1/index.html) > [Руководство по использованию](http://docs.wagtail.io/en/v1.13.1/topics/index.html) > Разрешения /////////////////// [Редактировать на GitHub](https://github.com/wagtail/wagtail/blob/b0be9edf510ce414c4050195817094b9261bc945/docs/topics/permissions.rst)
___

Разрешения
----------
В Wagtail адаптирована и расширена [система разрешений Django](https://docs.djangoproject.com/en/1.10/topics/auth/default/#topic-authorization) для полноценной работы в создании контента для веб-сайта. Права изменения контента, разделение на группы и наделение групп определенными правами в ограниченной области. Так-же контроль прав на редактирование разных веб-сайтов в рамках одного проекта. Разрешения на редактирование контента могут быть редактированы в настройках через Администраторскую панель(Админ-панель) в разделе Группы.

Разрешения страниц
------------------
В структуре сайта разрешения могут быть включены в любом разделе. Они также будут распространятся на подразделы. Например сайт имеет следующую структуру:

    MegaCorp/
        About us
        Offices/
            UK
            France
            Germany
группа наделенная правами на редактирование раздела «Offices» так-же автоматически получает права на редактирование подразделов «UK», «France», «Germany». Разрешения могут быть настроены глобально, предоставив их на редактирование корневого раздела, корневой раздел не может быть удален, таким образом группа получает права на редактирование текущих разделов, а так-же всех разделов которые будут добавлены в текущий корневой раздел в будущем.

Всякий раз когда пользователь создает страницу через Админ-панель, он назначается владельцем этой страницы. Пользователь с правами «add» имеет возможность редактировать свои страницы так-же как и добавлять новые. Создание страниц, как правило, является итеративным процессом, при котором пользователь создает своего рода черновик до его публикации - давая пользователям возможность создавать черновик, но не позволяя им впоследствии редактировать его, было бы не очень полезно. Возможность редактирования страницы также подразумевает возможность ее удаления; в отличие от стандартной модели разрешения Django, тут нет определенного разрешения «delete».

Полный набор доступных типов разрешений следующий:

 - **Add** - позволяет создавать новые подстраницы в текущей странице (при условии, что модель страницы это разрешает - см. [Права редактирования родительской страницы / подстраницы](http://docs.wagtail.io/en/v1.13.1/topics/pages.html#page-type-business-rules)) так-же  редактировать и удалять страницы, принадлежащие текущему пользователю. Опубликованные страницы не могут быть удалены, если пользователь также не имеет «publish» права.
 - **Edit** - предоставляет возможность редактировать и удалять текущую страницу и любые страницы под ней, независимо от прав собственности. Пользователь с разрешением «edit» не может создавать новые страницы, но может редактировать уже существующие. Опубликованные страницы не могут быть удалены, если пользователь также не имеет «publish» права.
 - **Publish** - предоставляет возможность публиковать и отменять публикацию той или иной страницы и / или ее подстраниц. Пользователь без разрешения на публикацию не может непосредственно вносить изменения, которые видны в последствии посетителям веб-сайта; вместо этого они должны представить свои изменения для модерации, при котором отправляется уведомление пользователям с разрешением на публикацию. Разрешение публикации не зависит от разрешения на редактирование; пользователь, имеющий только разрешение на публикацию, не сможет самостоятельно редактировать свои права (наделять себя другими правами).
 - **Bulk delete** - позволяет пользователю удалять страницы, у которых есть подстраницы, за одну операцию. Без этого разрешения пользователь должен удалить сначала подстраницы отдельно перед удалением текущей. Это защита от случайного удаления. Это разрешение должно использоваться в сочетании с разрешениями «add» и «edit», поскольку само по себе оно не предоставляет никаких прав на удаление; это своего рода «ярлык» на разрешения, которые пользователь уже имеет. Например, пользователь с правами «add» и «bulk delete» сможет выполнять удаление, если все затронутые страницы принадлежат этому пользователю и не были опубликованы.
 - **Lock** - предоставляет возможность блокировки или разблокировки текущей страницы (и всех подстраниц) на редактирования, не позволяя пользователям делать какие-либо дальнейшие изменения.

Черновики могут просматриваться только в том случае, если пользователь имеет разрешение «edit» или «publish».

Разрешения на изображения и документы
-------------------------------------
Правила разрешений для изображений и документов работают таким-же образом как и для страниц. Изображения и документы считаются «принадлежащими» пользователям, которые их загрузили; пользователи с разрешением «add» также имеют возможность редактировать элементы, которые добавили они и являются владельцами данных элементов; и удаление считается эквивалентным редактированию, а не определенному типу разрешения.

Доступ к определенным наборам изображений и документов может контролироваться путем создания коллекций. По умолчанию все изображения и документы принадлежат коллекции «root», но новые коллекции могут быть созданы через область Админ-интерфейса «Настройки -> Коллекции». Разрешения, установленные в «root», применяются ко всем коллекциям, поэтому пользователь с разрешением «edit» для изображений в «root» может редактировать все изображения; разрешения, установленные для других коллекций, применяются только к той коллекции.
