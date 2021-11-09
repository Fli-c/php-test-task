## Задание

Необходимо реализовать систему для редактирования json-документов методом PATCH.
Клиент системы должен иметь возможность создать пустой черновик документа.
Пока документ находится в статусе черновик, его можно редактировать сколько угодно раз.
Черновик документа можно опубликовать.
После публикации документ больше редактировать нельзя.

## Требования

|                  |                             |
|------------------|-----------------------------|
| Время исполнения | 8-16ч опытным разработчиком |
| Фреймворк        | Laravel или Symfony         |
| Сторонние пакеты | любые                       |
| PHP              | >=7.4                       |
| DB               | MySQL                       |

1. Разместить код в любом доступном git-резпозитории.
2. В файле `README.md` описать как запустить проект.
3. Соблюдать единый code-style на протяжении всего проекта.
4. Обязательна документация для каждого метода, класса и поля. Указание типов обязательно.
5. Отчет в виде затраченного времени, полнота исполнения задания, а также, возникшие проблемы сложности и их решения, пожелания, комментарий и пр.

## API

- `POST /api/v1/document/` - создаем черновик документа
- `GET /api/v1/document/{id}` - получить документ по id
- `PATCH /api/v1/document/{id}` - редактировать документ
- `POST /api/v1/document/{id}/publish` - опубликовать документ
- `GET /api/v1/document/?page=1&perPage=20` - получить список документов с пагинацией, сортировка в последние созданные сверху.

Условия:

- Если документ не найден, то в ответе возвращается 404 код.
- При попытке редактирования документа, который уже опубликован, должно возвращаться 400.
- Попытка опубликовать уже опубликованный документ  возвращает 200.
- Все запросы на конкретный документ возвращают этот документ. [JsonSchema ответа с документом](document-response.json).
- Список документов возвращается в виде массива документов и значений пагинации. [JsonSchema списка документов](document-list-response.json).
- Запрос `PATCH` отправляется с телом json в соответсвующей иерархии документа, все поля, кроме `payload` игнорируются. Если `payload` не передан, то ответ 400.

### Объект документа

```js
document = {
  id: "some-uuid-string",
  status: "draft|published",
  payload: Object,
  created_at: "iso-8601 date time with time zone",
  updated_at: "iso-8601 date time with time zone"
}
```

[JsonSchema для документа](document.json)

## Патчинг документа

Патчинг проводится согласно [RFC-7396](https://tools.ietf.org/html/rfc7396).

## Пример работы

### 1. Клиент делает запрос на создание документа

Запрос:

```http
POST /api/v1/document HTTP/1.1
accept: application/json
```

Ответ:

```http
HTTP/1.1 200 OK
content-type: application/json

{
    "document": {
        "id": "718ce61b-a669-45a6-8f31-32ba41f94784",
        "status": "draft",
        "payload": {},
        "created_at": "2018-09-01 20:00:00+07:00",
        "updated_at": "2018-09-01 20:00:00+07:00"
    }
}
```

### 2. Клиент редактирует документ первый раз

Запрос:

```http
PATCH /api/v1/document/718ce61b-a669-45a6-8f31-32ba41f94784 HTTP/1.1
accept: application/json
content-type: application/json

{
    "document": {
        "payload": {
            "actor": "The fox",
            "meta": {
                "type": "quick",
                "color": "brown"
            },
            "actions": [
                {
                    "action": "jump over",
                    "actor": "lazy dog"
                }
            ]
        }
    }
}
```

Ответ:

```http
HTTP/1.1 200 OK
content-type: application/json

{
    "document": {
        "id": "718ce61b-a669-45a6-8f31-32ba41f94784",
        "status": "draft",
        "payload": {
            "actor": "The fox",
            "meta": {
                "type": "quick",
                "color": "brown"
            },
            "actions": [
                {
                    "action": "jump over",
                    "actor": "lazy dog"
                }
            ]
        },
        "created_at": "2018-09-01 20:00:00+07:00",
        "updated_at": "2018-09-01 20:01:00+07:00"
    }
}
```

### 3. Клиент редактирует документ

Запрос:

```http
PATCH /api/v1/document/718ce61b-a669-45a6-8f31-32ba41f94784 HTTP/1.1
accept: application/json
content-type: application/json

{
    "document": {
        "payload": {
            "meta": {
                "type": "cunning",
                "color": null
            },
            "actions": [
                {
                    "action": "eat",
                    "actor": "blob"
                },
                {
                    "action": "run away"
                }
            ]
        }
    }
}
```

Ответ:

```http
HTTP/1.1 200 OK
content-type: application/json

{
    "document": {
        "id": "718ce61b-a669-45a6-8f31-32ba41f94784",
        "status": "draft",
        "payload": {
            "actor": "The fox",
            "meta": {
                "type": "cunning"
            },
            "actions": [
                {
                    "action": "eat",
                    "actor": "blob"
                },
                {
                    "action": "run away"
                }
            ]
        },
        "created_at": "2018-09-01 20:00:00+07:00",
        "updated_at": "2018-09-01 20:02:00+07:00"
    }
}
```

### 4. Клиент публикует документ

Запрос:

```http
POST /api/v1/document/718ce61b-a669-45a6-8f31-32ba41f94784/publish HTTP/1.1
accept: application/json
```

Ответ:

```http
HTTP/1.1 200 OK
content-type: application/json

{
    "document": {
        "id": "718ce61b-a669-45a6-8f31-32ba41f94784",
        "status": "published",
        "payload": {
            "actor": "The fox",
            "meta": {
                "type": "cunning"
            },
            "actions": [
                {
                    "action": "eat",
                    "actor": "blob"
                },
                {
                    "action": "run away"
                }
            ]
        },
        "created_at": "2018-09-01 20:00:00+07:00",
        "updated_at": "2018-09-01 20:03:00+07:00"
    }
}
```

### 5. Клиент получает запись в списке

Запрос:

```http
GET /api/v1/document/?page=1 HTTP/1.1
accept: application/json
```

Ответ:

```http
HTTP/1.1 200 OK
content-type: application/json

{
    "document": [
        {
            "id": "718ce61b-a669-45a6-8f31-32ba41f94784",
            "status": "published",
            "payload": {
                "actor": "The fox",
                "meta": {
                    "type": "cunning"
                },
                "actions": [
                    {
                        "action": "eat",
                        "actor": "blob"
                    },
                    {
                        "action": "run away"
                    }
                ]
            },
            "created_at": "2018-09-01 20:00:00+07:00",
            "updated_at": "2018-09-01 20:03:00+07:00"
        }
    ],
    "pagination": {
        "page": 1,
        "perPage": 20,
        "total": 1
    }
}
```

## Дополнительные задания

Дополнительные задания не являются обязательными, однако, если вы не будете испытывать сложности с их реализацией, то, вероятно, вы опытный разработчик. Если же сложности все-таки возникают, то значит есть повод их преодолеть и получить новые знания. Полнота и способ исполнения дополнительных заданий - целиком продукт вашего творчества, ведь программист - творческая профессия.

### Написать unit/feature тесты

Необходимо описать набор тестов в PHPUnit, для того чтобы убедиться что ваше приложение работоспособно.

### Декомпозиция задач

Составить план работ/модулей которые надо реализовать. Дать оценку времени, которое будет затрачено на каждый из пунктов. По окончанию реализации задачи записать сколько реально времени было потрачено.

В итоге получить примерно такую таблицу:

| #  | Задача               | Оценка | Затрачено | Комментарий               |
|:--:|:---------------------|:------:|:---------:|:--------------------------|
| 1  | Настройка окружения  | 1ч     | 40м       | Нашел хорошую инструкцию  |
| 2  | Установка фреймворка | 20м    | 30м       | Забыл установить composer |
| 3  | ...                  | ...    | ...       | ...                       |

### Docker или Podman

Завернуть ваше приложение в Docker/Podman контейнер. Написать скрипт разворачивания приложения одной командой. Например, при помощи docker-compose.

### Добавить фронт

- Добавить возможность получения списка документов с пагинатором по пути `/`
- Просмотр конкретного документа по пути `/document/{id}`

Внешний вид нас не интересует - поэтому не стоит пытаться сделать все идеально. Важен способ реализации. Лучше использовать готовые решения и фреймворки. Ограничений нет - полная свобода творчества.

### Реализовать патчинг самостоятельно

В вашем приложении вы могли использовать готовый модуль патча json. Если это так, то попробуйте реализовать алгоритм патчинга самостоятельно.

### Добавить аутентификацию/авторизацию

Реализовать авторизацию без пароля `POST /api/v1/login`\
Запрос:

```http
POST /api/v1/login HTTP/1.1
accept: application/json
content-type: application/json

{
    "login": "root"
}
```

Ответ:

```http
HTTP/1.1 200 OK
content-type: application/json

{
    "user": "root",
    "token": "q56lVCW9aIW6Gs01F5N9raaQordCb8HW",
    "until": 1537352295
}
```

`token` - это случайная строка из символов.\
`until` - это unix timestamp до которого действует токен.

Аутентификация должна проходить по заголовку в запросе. Например:

```http
GET /api/v1/document/718ce61b-a669-45a6-8f31-32ba41f94784 HTTP/1.1
accept: application/json
Authorization: bearer q56lVCW9aIW6Gs01F5N9raaQordCb8HW
```

Требования:

1. Каждый раз когда пользователь авторизуется - он получает новый токен.
2. Если пользователь шлет любой запрос с несуществующим или просроченным токеном - должен вернуться ответ с кодом 401.
3. Аноним может видеть список опубликованных документов, а также загружать конкретный опубликованный документ.
4. Аноним при попытке обратиться к PATCH и POST запросам будет получать ошибку 401.
5. Документ может создать только пользователь.
6. Редактировать и опубликовать документ может только пользователь создавший его.
7. Пользователь в списке документов видит только свои неопубликованные документы и опубликованные документы других пользователей, но не видит неопубликованные документы других.
8. Пользователь при попытке обратиться к чужому, неопубликованному документу получает 403.
9. Время действия токена - 1 час.

### Учесть потребление памяти

Как будет вести себя ваше приложение, если запрошенный список документов не помещается в php memory_limit? Если есть проблемы - надо их устранить.
