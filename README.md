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

Запустим кластер Elastic:
```sh
docker run --name elastic-demo -p 9200:9200 -e "discovery.type=single-node" -e "xpack.security.enabled=false" elasticsearch:9.2.4
```

Переменные:
- `discovery.type=single-node` - кластер, состоящий из одного узла
- `xpack.security.enabled=false` - отключаем `https`

Проверим:

```sh
curl http://localhost:9200

{
  "name" : "64f473f36498",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "YgotWteFQsGkxSw_FdhiZw",
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
```


## API Elasticsearch

Взаимодействие с Elasticsearch происходит по http протоколу командами `GET`, `PUT`, `POST` и другими в зависимости от выполняемой операции.
Мы можем обращаться к Elasticsearch с помощью утилиты командной строки `curl` или через удобный графисеский иннерфейс как `Postman`.


### Индекс

Создание индекса...
