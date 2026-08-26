---
title: "MIET Schedule API v1: публичная документация"
description: "Справочник read-only API v1 расписания МИЭТ: авторизация, endpoints, параметры, abilities, ответы и ошибки."
lang: ru
robots: index, follow
---

# MIET Schedule API v1

API предоставляет read-only доступ к опубликованному расписанию, справочникам и изменениям. Он не предназначен для изменения данных.

Проект и API не являются официальным сервисом МИЭТ.

## Базовые адреса

Подставьте публичный домен сервиса вместо `https://<PUBLIC-HOST>`.

| Назначение | Базовый URL |
|---|---|
| Production API | `https://<PUBLIC-HOST>/api/v1` |
| Playground | `https://<PUBLIC-HOST>/api/playground/v1` |
| OpenAPI 3.0.3 | `https://<PUBLIC-HOST>/api/v1/openapi.json` |
| HTML-документация | `https://<PUBLIC-HOST>/api/docs` |

Playground предоставляет тот же read-only набор маршрутов без bearer-токена, но ограничивает частоту запросов. Он предназначен для знакомства, а не для production-нагрузки.

## Авторизация

Production API принимает bearer-токен:

```http
Authorization: Bearer <API_TOKEN>
Accept: application/json
```

Не публикуйте токен, не добавляйте его в URL и не храните в клиентском коде, доступном пользователям.

Чтобы получить API Токен - обратитесь к администрации сайта (контакты указаны в разделе "о сайте")
## Abilities

Токен может разрешать все read-only разделы или отдельные области:

| Ability | Назначение |
|---|---|
| `read:*` | Все read-only разделы |
| `status:read` | Статус, семестр и время пар |
| `groups:read` | Группы |
| `teachers:read` | Преподаватели |
| `rooms:read` | Аудитории |
| `free_rooms:read` | Свободные аудитории |
| `schedule:read` | Расписание |
| `changes:read` | Изменения расписания |
| `bulk:read` | Bulk-чтение расписания |

Некоторые endpoints требуют сочетание двух abilities.

## Общий формат ответа

Успешный ответ содержит `data` и при необходимости `meta`:

```json
{
  "data": {},
  "meta": {}
}
```

Ошибка имеет единый объект `error`:

```json
{
  "error": {
    "code": "validation_error",
    "message": "Проверьте параметры запроса.",
    "details": {}
  }
}
```

Поле `details` присутствует не у каждой ошибки. Обрабатывайте как минимум статусы `401`, `403`, `404`, `422`, `429` и ошибки сервера, не полагаясь только на текст сообщения.

## Статус и справочники времени

| Метод | Endpoint | Ability |
|---|---|---|
| GET | `/status` | `status:read` |
| GET | `/semester` | `status:read` |
| GET | `/data-status` | `status:read` |
| GET | `/pairs` | `status:read` |

Ответы используют часовой пояс `Europe/Moscow`. Статус сообщает, в частности, версию API и время обновления расписания.

## Группы

| Метод | Endpoint | Ability |
|---|---|---|
| GET | `/groups` | `groups:read` |
| GET | `/groups/search` | `groups:read` |
| GET | `/groups/{group}/schedule` | `groups:read`, `schedule:read` |
| GET | `/groups/{group}/schedule/today` | `groups:read`, `schedule:read` |
| GET | `/groups/{group}/schedule/tomorrow` | `groups:read`, `schedule:read` |
| GET | `/groups/{group}/schedule/week` | `groups:read`, `schedule:read` |
| GET | `/groups/{group}/current` | `groups:read`, `schedule:read` |
| GET | `/groups/{group}/next` | `groups:read`, `schedule:read` |

Используйте идентификатор `id` из каталога групп и URL-кодируйте path parameter.

## Преподаватели

| Метод | Endpoint | Ability |
|---|---|---|
| GET | `/teachers` | `teachers:read` |
| GET | `/teachers/search` | `teachers:read` |
| GET | `/teachers/{id}/schedule` | `teachers:read`, `schedule:read` |
| GET | `/teachers/{id}/current` | `teachers:read`, `schedule:read` |
| GET | `/teachers/{id}/next` | `teachers:read`, `schedule:read` |

Для `{id}` используйте непрозрачный идентификатор из каталога преподавателей. Не формируйте его самостоятельно.

## Аудитории

| Метод | Endpoint | Ability |
|---|---|---|
| GET | `/rooms` | `rooms:read` |
| GET | `/rooms/search` | `rooms:read` |
| GET | `/rooms/{room}/schedule` | `rooms:read`, `schedule:read` |
| GET | `/rooms/{room}/current` | `rooms:read`, `schedule:read` |
| GET | `/rooms/{room}/next` | `rooms:read`, `schedule:read` |
| GET | `/rooms/{room}/availability` | `rooms:read`, `schedule:read` |

## Свободные аудитории и общий поиск

| Метод | Endpoint | Ability |
|---|---|---|
| GET | `/free-rooms` | `free_rooms:read` |
| GET | `/free-rooms/now` | `free_rooms:read` |
| GET | `/free-rooms/next` | `free_rooms:read` |
| GET | `/search` | `groups:read`, `teachers:read`, `rooms:read` |

## Изменения расписания

| Метод | Endpoint | Ability |
|---|---|---|
| GET | `/changes` | `changes:read` |
| GET | `/groups/{group}/changes` | `changes:read` |
| GET | `/teachers/{id}/changes` | `changes:read` |
| GET | `/rooms/{room}/changes` | `changes:read` |
| GET | `/schedule-versions/{id}/diff` | `changes:read` |

## Bulk-чтение

Bulk endpoints выполняют read-only POST-запросы и принимают не более 10 идентификаторов соответствующего типа за запрос.

| Метод | Endpoint | Abilities | Поле массива |
|---|---|---|---|
| POST | `/schedules/groups/bulk` | `bulk:read`, `schedule:read` | `groups` |
| POST | `/schedules/teachers/bulk` | `bulk:read`, `schedule:read` | `teachers` |
| POST | `/schedules/rooms/bulk` | `bulk:read`, `schedule:read` | `rooms` |

Передавайте `Content-Type: application/json`.

## Query parameters

### Каталоги

| Параметр | Формат | Ограничение |
|---|---|---|
| `q` | string | До 80 символов |
| `page` | integer | От 1 |
| `per_page` | integer | От 1 до 100 |

### Расписание

| Параметр | Формат | Описание |
|---|---|---|
| `date` | `YYYY-MM-DD` | Один день |
| `from` | `YYYY-MM-DD` | Начало диапазона |
| `to` | `YYYY-MM-DD` | Конец диапазона |
| `week` | integer `0..3` | Индекс недели |
| `group_by` | `day` или `none` | Группировка элементов |

Если указаны `from` и `to`, дата окончания не может быть раньше начала, а диапазон не может превышать 31 день.

### Свободные аудитории

| Параметр | Формат | Описание |
|---|---|---|
| `date` | `YYYY-MM-DD` | Дата |
| `pair` | integer `1..8` | Номер пары |
| `q` | string | Фильтр по названию, до 80 символов |
| `exclude_virtual` | boolean (`0` или `1`) | Исключить виртуальные помещения |

### Общий поиск

| Параметр | Формат | Ограничение |
|---|---|---|
| `q` | string | Обязательный, до 80 символов |
| `limit` | integer | От 1 до 30 |

## Кеширование и актуальность

Некоторые ответы содержат `ETag`, `Last-Modified` и `Cache-Control`. Клиентам рекомендуется выполнять условные запросы с `If-None-Match` или `If-Modified-Since` и корректно обрабатывать `304 Not Modified`.

В `meta` могут присутствовать время обновления, часовой пояс, признак неофициального сервиса и предупреждение о возможной неактуальности.

## Ограничение частоты запросов

Production-квоты зависят от выданного токена. При превышении лимита API возвращает `429`. Используйте кеширование, bulk endpoints и exponential backoff; не повторяйте запросы в плотном цикле.

## Версионирование

Текущая версия API — `v1`. Не удаляйте поддержку существующих полей на стороне клиента без проверки. Неизвестные поля следует игнорировать, чтобы клиент сохранял совместимость с расширением ответа.

Полные машиночитаемые сведения доступны в [OpenAPI JSON](/api/v1/openapi.json). Практические запросы приведены в [примерах API](api-examples.md).

---

[Документация](README.md) · [Сценарии](user-guide.md) · [PWA](pwa.md) · [Telegram](telegram.md) · [API](api.md) · [Примеры API](api-examples.md) · [Ресурсы](resources.md)
