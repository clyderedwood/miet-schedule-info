---
title: "Примеры MIET Schedule API v1"
description: "Примеры curl, JavaScript и PHP для поиска групп, чтения расписания и bulk-запросов MIET Schedule API v1."
lang: ru
robots: index, follow
---

# Примеры использования API

В примерах используются placeholders:

- `https://<PUBLIC-HOST>` — публичный домен сервиса;
- `<API_TOKEN>` — выданный bearer-токен;
- `<GROUP-ID>`, `<TEACHER-ID>`, `<ROOM-ID>` — идентификаторы из соответствующих каталогов.

Не вставляйте реальный токен в публичный репозиторий, browser bundle, URL или снимок экрана.

## Проверить статус

```bash
curl --request GET \
  --url "https://<PUBLIC-HOST>/api/v1/status" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <API_TOKEN>"
```

Структура успешного ответа:

```json
{
  "data": {
    "service": "miet-schedule",
    "version": "v1",
    "timezone": "Europe/Moscow",
    "schedule_updated_at": "2026-01-15T10:00:00+03:00",
    "official": false,
    "mode": "<SERVICE-MODE>"
  }
}
```

Значения времени и режима приведены только как пример.

## Найти группу

```bash
curl --get "https://<PUBLIC-HOST>/api/v1/groups/search" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <API_TOKEN>" \
  --data-urlencode "q=<GROUP-NAME>" \
  --data-urlencode "per_page=10"
```

Структура ответа каталога:

```json
{
  "data": [
    {
      "id": "<GROUP-ID>",
      "name": "<GROUP-NAME>"
    }
  ],
  "meta": {
    "pagination": {
      "page": 1,
      "per_page": 10,
      "total": 1,
      "last_page": 1
    }
  }
}
```

## Получить расписание группы за период

Path parameter должен быть URL-кодирован.

```bash
curl --get "https://<PUBLIC-HOST>/api/v1/groups/<GROUP-ID>/schedule" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <API_TOKEN>" \
  --data-urlencode "from=2026-09-01" \
  --data-urlencode "to=2026-09-07" \
  --data-urlencode "group_by=none"
```

Сокращённая структура ответа:

```json
{
  "data": {
    "entity": {
      "type": "group",
      "id": "<GROUP-ID>",
      "name": "<GROUP-NAME>"
    },
    "period": {
      "from": "2026-09-01",
      "to": "2026-09-07",
      "timezone": "Europe/Moscow"
    },
    "items": [
      {
        "date": "2026-09-01",
        "day_number": 2,
        "pair": 1,
        "time_start": "<HH:MM>",
        "time_end": "<HH:MM>",
        "subject": "<SUBJECT>",
        "class_type": null,
        "teacher": {
          "id": "<TEACHER-ID>",
          "name": "<TEACHER-NAME>"
        },
        "room": {
          "name": "<ROOM-NAME>"
        },
        "groups": [
          {
            "name": "<GROUP-NAME>"
          }
        ],
        "changed_recently": false
      }
    ]
  },
  "meta": {
    "schedule_updated_at": "<ISO-8601-DATETIME>",
    "official": false,
    "timezone": "Europe/Moscow"
  }
}
```

Поля отдельных элементов могут быть `null`, а `items` может быть пустым.

## Выполнить общий поиск

```bash
curl --get "https://<PUBLIC-HOST>/api/v1/search" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <API_TOKEN>" \
  --data-urlencode "q=<SEARCH-TEXT>" \
  --data-urlencode "limit=10"
```

## Найти свободные аудитории

```bash
curl --get "https://<PUBLIC-HOST>/api/v1/free-rooms" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <API_TOKEN>" \
  --data-urlencode "date=2026-09-01" \
  --data-urlencode "pair=2" \
  --data-urlencode "exclude_virtual=1"
```

## Получить несколько расписаний одним запросом

```bash
curl --request POST \
  --url "https://<PUBLIC-HOST>/api/v1/schedules/groups/bulk" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <API_TOKEN>" \
  --header "Content-Type: application/json" \
  --data '{
    "groups": ["<GROUP-ID-1>", "<GROUP-ID-2>"],
    "date": "2026-09-01",
    "group_by": "none"
  }'
```

## JavaScript на сервере

Не используйте этот пример в публичном browser bundle: токен должен оставаться на доверенной серверной стороне.

```js
const response = await fetch('https://<PUBLIC-HOST>/api/v1/groups?per_page=20', {
    headers: {
        Accept: 'application/json',
        Authorization: `Bearer ${process.env.MIET_SCHEDULE_API_TOKEN}`,
    },
});

if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error?.message ?? `HTTP ${response.status}`);
}

const payload = await response.json();
console.log(payload.data);
```

## PHP

```php
<?php

$token = getenv('MIET_SCHEDULE_API_TOKEN');
$url = 'https://<PUBLIC-HOST>/api/v1/pairs';

$context = stream_context_create([
    'http' => [
        'method' => 'GET',
        'header' => [
            'Accept: application/json',
            'Authorization: Bearer '.$token,
        ],
        'ignore_errors' => true,
    ],
]);

$body = file_get_contents($url, false, $context);
$payload = json_decode((string) $body, true, flags: JSON_THROW_ON_ERROR);
```

## Обработать кеширование

Сохраните полученный `ETag` и передайте его в следующем запросе:

```http
If-None-Match: "<ETAG>"
```

При ответе `304 Not Modified` используйте локально сохранённое представление и не ожидайте JSON body.

## Обработать ошибку

```json
{
  "error": {
    "code": "not_found",
    "message": "Группа не найдена."
  }
}
```

Клиент должен учитывать HTTP status, `error.code` и возможность появления дополнительных полей.

---

[Документация](README.md) · [Сценарии](user-guide.md) · [PWA](pwa.md) · [Telegram](telegram.md) · [API](api.md) · [Примеры API](api-examples.md) · [Ресурсы](resources.md)
