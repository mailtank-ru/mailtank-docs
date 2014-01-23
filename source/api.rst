API
===

.. contents::
    :local:
    :backlinks: top

Введение
--------
Проект предоставляет RESTful API для управления подписчиками, создания шаблонов
и выполнения рассылок.  Все ответы API имеют `Content-Type: application/json` и
содержат валидный JSON (за исключением ответов c кодом 204). Все запросы к API
также должны иметь `Content-Type: application/json`.

Авторизация
-----------
Авторизация производится при помощи ключа доступа, который клиент указывает в
HTTP-заголовке `X-Auth-Token`.  Ключ предоставляет доступ к ресурсам одного
проекта.

Ресурсы
-------
Проект
++++++++++++++++++++

Информация о проекте содержит следующие поля:

* ``name``: имя проекта Mailtank
* ``from_email``: e-mail адрес, с которого отправляются письма проекта

.. http:get:: /project
    
    Возвращает иформацию о текущем проекте

    .. sourcecode:: http

        GET /project HTTP/1.1

    .. sourcecode:: http

        HTTP/1.0 200 OK

        {
            "name": "project name",
            "from_email": "no-reply@project.domain"
        }

Подписчики
++++++++++

Информация о подписчике содержит следующие поля:

* ``does_email_exist`` (read-only): имеет истинное значение, если Mailtank
  не получал уведомлений от внешных систем о том, что данный e-mail не
  существует;
* ``email``: уникальный в рамках проекта e-mail;
* ``id``: уникальный в рамках проекта идентификатор подписчика;
* ``properties``: словарь свойств подписчика, которые доступны в шаблоне рассылки;
* ``tags``: список тегов подписчика;
* ``url`` (read-only): resource URI.

.. http:get:: /subscribers/?page=(int:page)

    Возвращает постраничный список подписчиков проекта.
   
    :query int page: (опционально) номер запрашиваемой страницы

    .. sourcecode:: http

        GET /subscribers/ HTTP/1.1

    .. sourcecode:: http

        HTTP/1.0 200 OK

        {
            "objects": [
                {
                    "does_email_exist": true,
                    "email": "james@doe.com",
                    "id": "1",
                    "properties": {
                        "age": 21,
                        "first_name": "James",
                        "last_name": "Doe"
                    },
                    "tags": [
                        "test-tag"
                    ],
                    "url": "/subscribers/1"
                },
                {
                    "does_email_exist": true,
                    "email": "john@doe.com",
                    "id": "192f56eb9b",
                    "properties": {},
                    "tags": [],
                    "url": "/subscribers/192f56eb9b"
                }
            ],
            "total": 2,
            "page": 1,
            "pages_total": 1
        }

.. http:post:: /subscribers/

    Создаёт нового подписчика.

    :jsonparam string email: e-mail подписчика. Должен быть уникальным в рамках
                             проекта.
    :jsonparam string id: (опционально) идентификатор подписчика. Должен быть
                          уникальным в рамках проекта. Будет сгенерирован
                          системой автоматически, если не указан в запросе.
    :jsonparam dict properties: (опционально) словарь свойств подписчика, ключи
                                которого являются строками, а значения --
                                строками или числами
    :jsonparam list tags: (опционально) список тегов подписчика
    
    Для создания подписчика достаточно указать e-mail:

    .. sourcecode:: http

        POST /subscribers/ HTTP/1.1

        {
            "email": "john@doe.com"
        }

    .. sourcecode:: http

        HTTP/1.0 201 CREATED

        {
            "does_email_exist": true,
            "email": "john@doe.com",
            "id": "192f56eb9b",
            "properties": {},
            "tags": [],
            "url": "/subscribers/192f56eb9b"
        }

    Нельзя создать второго подписчика с одним и тем же адресом:

    .. sourcecode:: http
        
        POST /subscribers/ HTTP/1.1

        {
            "email": "john@doe.com"
        }
   
    .. sourcecode:: http

        HTTP/1.0 400 BAD REQUEST

        {
            "email": ["Entry with such value already exists."]
        }

    При создании подписчика можно указать идентификатор, теги и свойства:

    .. sourcecode:: http

        POST /subscribers/ HTTP/1.1

        {
            "email": "james@doe.com",
            "id": "1",
            "properties": {
                "age": 21,
                "first_name": "James",
                "last_name": "Doe"
            },
            "tags": [
                "test-tag"
            ]
        }

    .. sourcecode:: http

        HTTP/1.0 201 CREATED

        {
            "does_email_exist": true,
            "email": "james@doe.com",
            "id": "1",
            "properties": {
                "age": 21,
                "first_name": "James",
                "last_name": "Doe"
            },
            "tags": [
                "test-tag"
            ],
            "url": "/subscribers/1"
        }

.. http:put:: /subscribers/(str:id)

    Обновляет данные подписчика.

    :jsonparam string email: (опционально) e-mail подписчика
    :jsonparam dict properties: (опционально) словарь свойств подписчика
    :jsonparam list tags: (опционально) список тегов подписчика
    
    .. sourcecode:: http

        PUT /subscribers/1 HTTP/1.1

        {
            "properties": {
                "age": 25,
                "first_name": "James",
                "last_name": "Doe",
                "sex": "M"
            },
            "tags": [
                "male",
                "yet-another-test-tag"
            ]
        }
    
    .. sourcecode:: http

        HTTP/1.0 200 OK

        {
            "does_email_exist": true,
            "email": "james@doe.com",
            "id": "1",
            "properties": {
                "age": 25,
                "first_name": "James",
                "last_name": "Doe",
                "sex": "M"
            },
            "tags": [
                "male",
                "yet-another-test-tag"
            ],
            "url": "/subscribers/1"
        }

.. http:get:: /subscribers/(str:id)

   Возвращает данные подписчика.
 
   .. sourcecode:: http

        GET /subscribers/1 HTTP/1.1

   .. sourcecode:: http

        HTTP/1.0 200 OK

        {
            "does_email_exist": true,
            "email": "james@doe.com",
            "id": "1",
            "properties": {
                "age": 21,
                "first_name": "James",
                "last_name": "Doe"
            },
            "tags": [
                "test-tag"
            ],
            "url": "/subscribers/1"
        }

.. http:delete:: /subscribers/(str:id)

   Удаляет подписчика.

.. http:patch:: /subscribers/

    Выполняет операцию над группой подписчиков. Доступные операции:

    * ``reassign_tag`` — переназначает тег новой группе подписчиков (тег станет
      принадлежать указанным подписчикам и только им). Параметры операции
      должны содержать:
     
      * ``tag`` — тег (строка);
      * ``subscriber`` — список идентификаторов подписчиков или ``"all"``
        для обозначения всех подписчиков проекта.

    :jsonparam string action: имя операции
    :jsonparam dict data: словарь, содержащий параметры операции

    После данного запроса тег ``good`` станет принадлежать исключительно
    подписчикам с идентификаторами ``1`` и ``192f56eb9b``. Со всех остальных
    подписчиков проекта он будет снят:

    .. sourcecode:: http
        
        PATCH /subscribers/ HTTP/1.1
        
        {
            "action": "reassign_tag",
            "data": {
                "subscribers": ["1", "192f56eb9b"], 
                "tag": "good"
            }
        }
   
    .. sourcecode:: http
        
        HTTP/1.0 204 NO CONTENT

Теги подписчиков
++++++++++++++++

Информация о теге содержит следующие поля:

* ``name``: имя тэга;

.. http:get:: /tags/?page=(int:page)&mask=(str:mask)

    Возвращает постраничный список тегов подписчиков проекта.

    :query int page: (опционально) номер запрашиваемой страницы
    :query str mask: (опционально) маска для тегов

    Если указана маска, метод вернет только теги, имя которых начинается
    с указанной маски.

    .. sourcecode:: http

        GET /tags/?mask=tag HTTP/1.1

    .. sourcecode:: http

        HTTP/1.0 200 OK

        {
            "objects": [
                {
                    "name": "tag-one"
                },
                {
                    "name": "tag-two"
                }
            ],
            "total": 2,
            "page": 1,
            "pages_total": 1
        }

.. http:get:: /tags/(str:name)/subscribers/?page=(int:page)

    Возвращает постраничный список подписчиков проекта, которым присвоен
    указанные тег.

    :query int page: (опционально) номер запрашиваемой страницы

    .. sourcecode:: http

        GET /tags/tag-one/subscribers/ HTTP/1.1

    .. sourcecode:: http

        HTTP/1.0 200 OK

        {
            "objects": [
                {
                    "does_email_exist": true,
                    "email": "james@doe.com",
                    "id": "1",
                    "properties": {
                        "age": 21,
                        "first_name": "James",
                        "last_name": "Doe"
                    },
                    "tags": [
                        "tag-one"
                    ],
                    "url": "/subscribers/1"
                },
                {
                    "does_email_exist": true,
                    "email": "john@doe.com",
                    "id": "192f56eb9b",
                    "properties": {},
                    "tags": [
                        "tag-one",
                        "tag-two"
                    ],
                    "url": "/subscribers/192f56eb9b"
                }
            ],
            "total": 2,
            "page": 1,
            "pages_total": 1
        }

Свойства подписчика
+++++++++++++++++++

.. http:get:: /subscribers/(str:id)/properties/

    Возвращает словарь свойств подписчика.

    .. sourcecode:: http

        GET /subscribers/1/properties/ HTTP/1.1

    .. sourcecode:: http

        HTTP/1.0 200 OK

        {
            "age": 25,
            "birthdate": "14.08.1990",
            "last_name": "Doe",
            "sex": "M"
        }

.. http:get:: /subscribers/(str:id)/properties/(string:name)
    
    Возвращает значение свойства подписчика.

    .. sourcecode:: http
        
        GET /subscribers/1/properties/first_name HTTP/1.1

    .. sourcecode:: http
        
        HTTP/1.0 200 OK

        {
            "value": "James"
        }

.. http:post:: /subscribers/(str:id)/properties/(string:name)

    Изменяет значение или создаёт новое свойство подписчика.

    .. sourcecode:: http

        POST /subscribers/1/properties/birthdate HTTP/1.1

        {
            "value": "14.08.1990"
        }

    .. sourcecode:: http

        HTTP/1.0 200 OK

        {
            "value": "14.08.1990"
        }

.. http:delete:: /subscribers/(str:id)/properties/(string:name)

    Удаляет свойство подписчика.

    .. sourcecode:: http

        DELETE /subscribers/1/properties/first_name HTTP/1.1

    .. sourcecode:: http

        HTTP/1.0 204 NO CONTENT

Рассылки
++++++++

Информация о рассылке содержит следующие поля:

* ``id``: целочисленный идентификатор рассылки;
* ``eta``: приблизительное время в секундах, через которое закончится рассылка
  или ``null``, если рассылка не исполняется в данный момент;
* ``status``: статус исполнения рассылки. Принимает следующие значения:

  * ``"FAILED"``, если рассылка завершена неуспешно;
  * ``"SUCCEEDED"``, если рассылка завершена успешно;
  * ``"ENQUEUED"``, если рассылка ожидает своей очереди.

* ``url``: resource URI.

.. http:get:: /mailings/

    Возвращает постраничный список рассылок проекта.
    
    .. sourcecode:: http
       
        GET /mailings/ HTTP/1.1

    .. sourcecode:: http
        
        HTTP/1.0 200 OK

        {
            "objects": [
                {
                    "eta": null,
                    "id": 13,
                    "status": "FAILED",
                    "url": "/mailings/13"
                },
                {
                    "eta": 0,
                    "id": 14,
                    "status": "SUCCEEDED",
                    "url": "/mailings/14"
                },
                {
                    "eta": 10,
                    "id": 15,
                    "status": "ENQUEUED",
                    "url": "/mailings/15"
                },
                {
                    "eta": 20,
                    "id": 16,
                    "status": "ENQUEUED",
                    "url": "/mailings/16"
                },
                {
                    "eta": null,
                    "id": 17,
                    "status": "ENQUEUED",
                    "url": "/mailings/17"
                }
            ],
            "total": 5,
            "page": 1,
            "pages_total": 1
        } 

.. http:post:: /mailings/

    Создаёт и выполняет рассылку.

    :jsonparam string layout_id: идентификатор шаблона, который будет
                                 использован для рассылки
    :jsonparam dict context: словарь, содержащий данные рассылки. Должен
                             удовлетворять структуре используемого шаблона
    :jsonparam dict target: словарь, задающий получателей рассылки.
                            Допустимы следующие поля:

        * ``unsubscribe_tags``: список тегов, которые буду сняты с подписчика
          при отписке. Поле обязательно, если ``context`` не содержит
          ``unsubscribe_link``, или не указан ``unsubscribe_property``.

        * ``unsubscribe_property``: свойство объекта subscriber, которое будет
          подставлено в ``unsubscribe_link``, если ``context`` не содержит
          ``unsubscribe_link``

        * ``tags_union``: (по умолчанию -- false) задаёт логику интерпретации
          списка тегов (пересечение или объединение, см. ниже);

        * ``tags_and_receivers_union``: (по умолчанию -- false) задаёт логику
          интерпретации наличия и списка тегов, и списка идентификаторов
          (пересечение или объединение, см.ниже).

        * ``subscribers``: список идентификаторов подписчиков, явно задающий
          группу подписчиков;

        * ``tags``: список тегов, задающий группу подписчиков следующим образом:
            
            * если ``tags_union`` имеет ложное значение -- в группу входят
              подписчики, каждый из которых имеет все из перечисленных тегов;
            * если ``tags_union`` имеет истинное значение -- в группу входят
              подписчики, каждый из которых имеет хотя бы один из перечисленных
              тегов.
        
        **Логика интерпретации полей**:

        * Если указаны поля и ``tags``, и ``subscribers``, то рассылка будет
          послана:

            * если ``tags_and_receivers_union`` имеет ложное значение --
              подписчикам, входящим в обе группы (пересечение);
            * если ``tags_and_receivers_union`` имеет истинное значение --
              подписчикам, входящим по меньше мере в одну из групп
              (объединение).
        * Если указано лишь одно из полей ``tags`` и ``subscribers``, рассылка
          будет послана подписчика из группы, заданной этим полем.
        * Словарь должен содержать по меньшей мере одно из полей ``subscribers``
          и ``tags``.

    :jsonparam list attachments: (опционально) список словарей, содержащих
                                 следующие поля:

        * ``name``: имя вложения;
        * ``data``: закодированное в BASE64 содержимое файла;
        * ``mimetype``: MIME-тип вложения.

        Суммарный объём файлов после декодирования не должен превышать 10 МБ.

    .. sourcecode:: http
        
        POST /mailings/ HTTP/1.1

        {
            "attachments": [
                {
                    "data": "SGVsbG8h",
                    "mimetype": "text/plain",
                    "name": "hello.txt"
                }
            ],
            "context": {
                "name": "Anton"
            },
            "layout_id": "56929c1607",
            "target": {
                "tags": [
                    "test-tag",
                    "male"
                ],
                "tags_and_receivers_union": true,
                "unsubscribe_tags": [
                    "test-tag"
                ]
            }
        }

    .. sourcecode:: http
        
        HTTP/1.0 201 CREATED

        {
            "eta": null,
            "id": 16,
            "status": "ENQUEUED",
            "url": "/mailings/16"
        }

.. http:get:: /mailings/(int:id)

    Возвращает данные рассылки.

    .. sourcecode:: http

        GET /mailings/17 HTTP/1.1

    .. sourcecode:: http

        HTTP/1.0 200 OK

        {
            "eta": 20,
            "id": 17,
            "status": "ENQUEUED",
            "url": "/mailings/17"
        }

Шаблоны
+++++++

.. http:post:: /layouts/

    Создаёт шаблон с единственным вариантом.

    :jsonparam string name: имя шаблона
    :jsonparam string markup: разметка единственного варианта шаблона
    :jsonparam string subject_markup: разметка темы шаблона

    :jsonparam string plaintext_markup: (опционально) разметка текстовой версии
                                        шаблона
    :jsonparam string base: (опционально) идентификатор родительского базового
                            шаблона
    :jsonparam string id: (опционально) идентификатор шаблона. Должен быть
                          уникальным в рамках проекта. Будет сгенерирован
                          системой автоматически, если не указан в запросе.

    .. sourcecode:: http
        
        POST /layouts/ HTTP/1.1

        {
            "markup": "<h1>Hello, {{ name }}!</h1><br><a href=\"{{ unsubscribe_link }}\">Unsubscribe</a>",
            "name": "Default",
            "subject_markup": "Just a hello"
        }

    .. sourcecode:: http

        HTTP/1.0 200 OK

        {
            "id": "56929c1607"
        }

.. http:post:: /base_layouts/

    Создаёт базовый шаблон.

    :jsonparam string name: имя
    :jsonparam string markup: разметка
    :jsonparam string id: (опционально) идентификатор базового шаблона. Должен
                          быть уникальным в рамках проекта. Будет сгенерирован
                          системой автоматически, если не указан в запросе.

    .. sourcecode:: http
        
        POST /base_layouts/ HTTP/1.1

        {
            "markup": "<div>{% block content %}<a href=\"{{ unsubscribe_link }}\">Unsubscribe</a>{% endblock %}</div>",
            "name": "Default"
        }

    .. sourcecode:: http

        HTTP/1.0 200 OK

        {
            "id": "271f93f45e"
        }

Отписки
+++++++

Информация об отписке содержит следующие поля:

* ``mailing_id``: целочисленный идентификатор рассылки;
* ``subscriber_id``: идентификатор подписчика;
* ``events``: список "отписочных" событий, которые подписчик произвёл в письме
              рассылки ``mailing_id``. События представляют собой словарь с
              двумя полями:
                * ``type``: тип события:
                            | ``"action"`` -- пользователь действительно отписался,
                            | ``"intent"`` -- пользователь просмотрел страницу отписки.
                * ``created_at``: время события в формате ISO;

.. http:get:: /unsubscribed/?since=(str:since)&page=(int:page)

    Возвращает постраничный список отписок подписчиков.
   
    :query int page: (опционально) номер запрашиваемой страницы;
    :query str since: (опционально) дата в формате ISO, начиная с которой перечислять
                      события отписки.

    .. sourcecode:: http
       
        GET /unsubscribed/ HTTP/1.1

    .. sourcecode:: http
        
        HTTP/1.0 200 OK

        {
            "objects": [
                {
                    "mailing_id": 1,
                    "subscriber_id": "1",
                    "events": [{
                        "type": "intent",
                        "created_at": "2013-10-03T10:49:01.658949"
                    }]
                },
                {
                    "mailing_id": 2,
                    "subscriber_id": "s139sw",
                    "events": [{
                        "type": "action",
                        "created_at": "2013-13-03T13:34:51.123901"
                    }]
                }
            ],
            "total": 2,
            "page": 1,
            "pages_total": 1
        }
