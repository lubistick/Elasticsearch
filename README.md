# Elasticsearch

Полнотекстовый поиск - такой поиск, который позволяет искать текст в документах по ключевым словам или фразам.
При полнотекстовом поиске анализируется содержимое текста и в результате выдаются более релевантные документы.

Elasticsearch - это хранилище документов с возможностью создавать полнотекстовые индексы для последующего поиска.
Чаще всего используется в качестве поискового движка.
Основан на библиотеке `Apache Lucene`.


## Основные понятия

- `Индексы` предназначены для группировки данных в логические структуры.
У нас может быть индекс по фильмам, индекс по товарам или какой-то еще.
- `Документы` - это записи в самом индексе.
- `Запросы` предназначены для поиска и фильтрации в индексе.


## Запуск

Запустим кластер Elasticsearch:

```sh
docker run --name elastic-demo -p 9200:9200 -e "discovery.type=single-node" -e "xpack.security.enabled=false" elasticsearch:9.2.4
```

Переменные:
- `discovery.type=single-node` - кластер, состоящий из одного узла
- `xpack.security.enabled=false` - отключаем `https`

Проверим:

```sh
curl -w "\nhttp status: %{http_code}\n" -X GET http://localhost:9200

{
  "name" : "da2405d0b7fa",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "_7UggJ-tQ4C21BLeFCSl8A",
  "version" : {
    "number" : "9.2.4",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "dfc5c38614c29a598e132c035b66160d3d350894",
    "build_date" : "2026-01-08T22:07:25.170027027Z",
    "build_snapshot" : false,
    "lucene_version" : "10.3.2",
    "minimum_wire_compatibility_version" : "8.19.0",
    "minimum_index_compatibility_version" : "8.0.0"
  },
  "tagline" : "You Know, for Search"
}

http status: 200
```

Опция `-w` (write-out) в `curl` позволяет выводить переменные после ответа сервера.
Мы использовали опцию, чтобы смотреть статус код ответа сервера.


## API Elasticsearch

Взаимодействие с Elasticsearch происходит по http протоколу командами `GET`, `PUT`, `POST` и другими в зависимости от выполняемой операции.
Мы можем обращаться к Elasticsearch с помощью утилиты командной строки `curl` или через удобный графический интерфейс как `Postman`.


### Индекс

Индекс в Elasticsearch представляет собой структуру подобную таблице в реляционной базе данных.
Индекс используется для хранения, поиска и анализа данных.
Индекс в Elasticsearch можно не создавать явно - если добавить документ в несуществующий индекс, Elasticsearch автоматически создаст этот индекс.
Однако, лучше создавать его самостоятельно, так мы получаем больший контроль над структурой данных,
можем уточнить, какие анализаторы будут использованы, какие поля стоит индексировать и многое другое.

Например, мы храним информацию о товарах в магазине.
Создадим индекс:

```sh
curl -w "\nhttp status: %{http_code}\n" -X PUT http://localhost:9200/product_index \
  -H "Content-Type: application/json" \
  -d '{
    "mappings": {
      "properties": {
        "title": {
          "type": "text",
          "analyzer": "russian"
        },
        "price": {
          "type": "float"
        },
        "available": {
          "type": "boolean"
        }
      }
    }
  }'


{"acknowledged":true,"shards_acknowledged":true,"index":"product_index"}
http status: 200
```

- индекс создается через `PUT` запрос
- название индекса указывается в адресной строке сразу после домена сервера, в нашем случае это `product_index`
- мы определили поля и типы данных в ключе `mappings` - название (`title`), цену (`price`) и доступность на складе (`available`)
- названия товаров хранятся на русском языке, поэтому выбираем русский анализатор (`russian`), он доступен из коробки Elasticsearch
- в ответе поле `acknowledged` со значением `true` означает, что индекс успешно создан


### Документ

Документы представляют собой записи в индексе.
Если сравнить с реляционными базами данных, то индекс в Elasticsearch аналогичен строке в таблице в реляционной БД, но со своими отличиями.
Теперь добавим документы в индекс:

```sh
curl -w "\nhttp status: %{http_code}\n" -X PUT http://localhost:9200/product_index/_doc/1 \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Беспроводные наушники",
    "price": 49.99,
    "available": true
  }'


{"_index":"product_index","_id":"1","_version":1,"result":"created","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":0,"_primary_term":1}
http status: 201
```

- документ добавляет в индекс с помощью `PUT` запроса
- в урле после указываем в какой индекс добавляем документ (`product_index`), затем пишем `_doc`, затем указываем ID документа (`1`)
- в теле запроса указываем поля товара, которые мы хотим сохранить
- в ответе поле `result` со значением `created` - был создан новый документ

Если мы отправим запрос по этому же пути, но изменим цену на `59.99`, то в ответе увидим `result` со значением `updated`:

```sh
curl -w "\nhttp status: %{http_code}\n" -X PUT http://localhost:9200/product_index/_doc/1 \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Беспроводные наушники",
    "price": 59.99,
    "available": true
  }'


{"_index":"product_index","_id":"1","_version":2,"result":"updated","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":1,"_primary_term":1}
http status: 200
```

В ответе также видим, что версия документа обновилась (поле `_version` со значением `2`).
Т.е. в результате запроса был обновлен существующий документ, а не создан новый.

Одна и та же команда может как создавать документ, так и обновлять его.
Это бывает удобно, однако у такого поведения есть и обратная сторона - мы можем случайно обновить или перезатереть какой-либо документ данными, сами того не заметив.

Выполним запрос на получение документа по его идентификатору:

```sh
curl -w "\nhttp status: %{http_code}\n" -X GET http://localhost:9200/product_index/_doc/1


{"_index":"product_index","_id":"1","_version":2,"_seq_no":1,"_primary_term":1,"found":true,"_source":{
    "title": "Беспроводные наушники",
    "price": 59.99,
    "available": true
  }}
http status: 200
```

Видим в ответе документ с идентификатором (`_id`) равным `1`.

Попробуем получить документ с идентификатором `2`:

```sh
curl -w "\nhttp status: %{http_code}\n" -X GET http://localhost:9200/product_index/_doc/2


{"_index":"product_index","_id":"2","found":false}
http status: 404
```

Документа с таким ID нет в нашем индексе, в ответ получили `found` со значением `false` и соответствующий http статус код.

Для удаления документа в индексе используется метод `DELETE`:

```sh
curl -w "\nhttp status: %{http_code}\n" -X DELETE http://localhost:9200/product_index/_doc/1


{"_index":"product_index","_id":"1","_version":3,"result":"deleted","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":2,"_primary_term":1}
http status: 200
```

Видим в ответе поле `result` со значением `deleted`.

Если попытаться удалить несуществующий документ, в ответ получили `found` со значением `false` как и в случае получения документа.
Попробуем еще раз удалить документ:

```sh
curl -w "\nhttp status: %{http_code}\n" -X DELETE http://localhost:9200/product_index/_doc/1


{"_index":"product_index","_id":"1","_version":1,"result":"not_found","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":3,"_primary_term":1}
http status: 404
```


## Поиск

Поиск - это ключевая функция Elasticsearch.
Для выполнения поиска по документам используется специальный эндпоинт `_search`:

```sh
curl -w "\nhttp status: %{http_code}\n" -X GET http://localhost:9200/product_index/_search


{"took":61,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":{"value":0,"relation":"eq"},"max_score":null,"hits":[]}}
http status: 200
```

В ответ мы получили все документы, находящиеся в индексе, т.к. мы не конкретизировали запрос, и все документы подходят под такие критерии поиска.
Сейчас у нас нет документов в индексе (ключ `hits` -> `total` -> `value` имеет значение `0`).

Добавим несколько десятков документов в индекс, для этого воспользуемся `_bulk` API.
Формат строго чередующийся - action line + данные, каждая запись на отдельной строке, в конце пустая строка:

```sh
curl -w "\nhttp status: %{http_code}\n" -X POST http://localhost:9200/_bulk \
  -H "Content-Type: application/json" \
  -d '
{"index": {"_index": "product_index", "_id": 2}}
{"title": "Беспроводные наушники", "price": 59.99, "available": true}
{"index": {"_index": "product_index", "_id": 3}}
{"title": "Беспроводные наушники1", "price": 109.99, "available": true}
'
```