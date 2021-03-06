# Домашнее задание к занятию "6.5. Elasticsearch"

## Задача 1

В этом задании вы потренируетесь в:
- установке elasticsearch
- первоначальном конфигурировании elastcisearch
- запуске elasticsearch в docker

Используя докер образ [centos:7](https://hub.docker.com/_/centos) как базовый и 
[документацию по установке и запуску Elastcisearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html):

- составьте Dockerfile-манифест для elasticsearch
- соберите docker-образ и сделайте `push` в ваш docker.io репозиторий
- запустите контейнер из получившегося образа и выполните запрос пути `/` c хост-машины

Требования к `elasticsearch.yml`:
- данные `path` должны сохраняться в `/var/lib`
- имя ноды должно быть `netology_test`

В ответе приведите:
- текст Dockerfile манифеста
- ссылку на образ в репозитории dockerhub
- ответ `elasticsearch` на запрос пути `/` в json виде

Подсказки:
- возможно вам понадобится установка пакета perl-Digest-SHA для корректной работы пакета shasum
- при сетевых проблемах внимательно изучите кластерные и сетевые настройки в elasticsearch.yml
- при некоторых проблемах вам поможет docker директива ulimit
- elasticsearch в логах обычно описывает проблему и пути ее решения

Далее мы будем работать с данным экземпляром elasticsearch.  

Ответ:  
Dockerfile:
```docker
FROM centos:7

ENV PATH=/usr/lib:/usr/lib/jvm/jre-11/bin:$PATH JAVA_HOME=/opt/elasticsearch-8.0.1/jdk/ ES_HOME=/opt/elasticsearch-8.0.1

RUN yum install wget -y \
    && yum install perl-Digest-SHA -y \
    && wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.0.1-linux-x86_64.tar.gz \
    && wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.0.1-linux-x86_64.tar.gz.sha512 \
    && shasum -a 512 -c elasticsearch-8.0.1-linux-x86_64.tar.gz.sha512 \ 
    && tar -C /opt -xzf elasticsearch-8.0.1-linux-x86_64.tar.gz \
    && rm elasticsearch-8.0.1-linux-x86_64.tar.gz\
    && rm elasticsearch-8.0.1-linux-x86_64.tar.gz.sha512 \
    && groupadd elasticsearch \
    && useradd -g elasticsearch elasticsearch \
    && mkdir /var/lib/elasticsearch \
    && mkdir /var/log/elasticsearch \
    && mkdir /opt/elasticsearch-8.0.1/snapshots \
    && chown elasticsearch:elasticsearch /var/lib/elasticsearch \
    && chown elasticsearch:elasticsearch /var/log/elasticsearch \
    && chown -R elasticsearch:elasticsearch /opt/elasticsearch-8.0.1/
    
ADD cfg/elasticsearch.yml /opt/elasticsearch-8.0.1/config/
    
USER elasticsearch
CMD ["/usr/sbin/init"]
CMD ["/opt/elasticsearch-8.0.1/bin/elasticsearch"]
```
Ссылка на образ: [gnoy4eg/elastic:latest](https://hub.docker.com/repository/docker/gnoy4eg/elastic)  

GET запрос к `/`:  
```bash
[gnoy@manjarokde-ws01 elasticsearch]$ docker exec elastic curl -X GET 'localhost:9200'
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   535  100   535    0     0   2867      0 --:--:-- --:--:-- --:--:--  2876
{
  "name" : "10edf5d585dc",
  "cluster_name" : "netology_test",
  "cluster_uuid" : "fpj0nufVSOSZbY-XSTsC8Q",
  "version" : {
    "number" : "8.0.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "801d9ccc7c2ee0f2cb121bbe22ab5af77a902372",
    "build_date" : "2022-02-24T13:55:40.601285296Z",
    "build_snapshot" : false,
    "lucene_version" : "9.0.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

## Задача 2

В этом задании вы научитесь:
- создавать и удалять индексы
- изучать состояние кластера
- обосновывать причину деградации доступности данных

Ознакомтесь с [документацией](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) 
и добавьте в `elasticsearch` 3 индекса, в соответствии со таблицей:

| Имя | Количество реплик | Количество шард |
|-----|-------------------|-----------------|
| ind-1| 0 | 1 |
| ind-2 | 1 | 2 |
| ind-3 | 2 | 4 |

Получите список индексов и их статусов, используя API и **приведите в ответе** на задание.

Получите состояние кластера `elasticsearch`, используя API.

Как вы думаете, почему часть индексов и кластер находится в состоянии yellow?

Удалите все индексы.

**Важно**

При проектировании кластера elasticsearch нужно корректно рассчитывать количество реплик и шард,
иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.  

Ответ:  
За спойлером спрятаны промежуточные действия, которые не нужно приводить в задаче как ответ.

<details>
  <summary>Console</summary>


Запускаем контейнер и заходим в него
```bash
[gnoy@manjarokde-ws01 elasticsearch]$ docker run -d --name elastic gnoy4eg/elastic:latest
c4a3f14fad0fb8d2097594d073aded8725b3e335ce2ba72099377247551e5f88
[gnoy@manjarokde-ws01 elasticsearch]$ docker exec -it elastic bash
[elasticsearch@c4a3f14fad0f /]$
```
Создаем индексы
```bash
[elasticsearch@c4a3f14fad0f /]$ curl -X PUT "localhost:9200/ind-1?pretty" -H 'Content-Type: application/json' -d'
> {
>   "settings": {
>     "index": {
>       "number_of_shards": 1,  
>       "number_of_replicas": 0 
>     }
>   }
> }
> '
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "ind-1"
}
[elasticsearch@c4a3f14fad0f /]$ curl -X PUT "localhost:9200/ind-2?pretty" -H 'Content-Type: application/json' -d'
> {
>   "settings": {
>     "index": {
>       "number_of_shards": 2,  
>       "number_of_replicas": 1 
>     }
>   }
> }
> '
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "ind-2"
}
[elasticsearch@c4a3f14fad0f /]$ curl -X PUT "localhost:9200/ind-3?pretty" -H 'Content-Type: application/json' -d'
> {
>   "settings": {
>     "index": {
>       "number_of_shards": 4,  
>       "number_of_replicas": 2 
>     }
>   }
> }
> '
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "ind-3"
}
```
</details>

Проверяем список индексов и их статус:
```bash
[elasticsearch@c4a3f14fad0f /]$ curl -X GET "localhost:9200/_cat/indices?v"
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   ind-1 WTinqxUEReKgjwe3P-UG3A   1   0          0            0       225b           225b
yellow open   ind-3 oqgIrjgBTPWsrGole2ltMA   4   2          0            0       900b           900b
yellow open   ind-2 Y3-U6r6jT12Tt-0mLTnJSw   2   1          0            0       450b           450b
[elasticsearch@81184aca62b7 /]$ curl -X GET 'localhost:9200/_cluster/health/ind-1?pretty'
{
  "cluster_name" : "netology_test",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 1,
  "active_shards" : 1,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
[elasticsearch@81184aca62b7 /]$ curl -X GET 'localhost:9200/_cluster/health/ind-2?pretty'
{
  "cluster_name" : "netology_test",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 2,
  "active_shards" : 2,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 2,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 44.44444444444444
}
[elasticsearch@81184aca62b7 /]$ curl -X GET 'localhost:9200/_cluster/health/ind-3?pretty'
{
  "cluster_name" : "netology_test",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 4,
  "active_shards" : 4,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 8,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 44.44444444444444
}
```
При выводе статуса (см. выше) индексов видим все наши созданные индексы. У них имеются статусы: зеленый (green) и желтый(yellow).  
Зеленый статус сообщает нам, что все шарды (в данном случае 1 шард) находится в состоянии assigned. Все реплики "в порядке" (их у нас указано 0)  
Желтый стстус сообщает нам, что  все primary шарды в состоянии assigned, но реплики (у нас их 1 и 2 шт. для  ind-2 и ind-3) находятся в состоянии unassigned. Произошло это из-за того, что у нас один сервер СУБД в кластере. Реплицировать некуда.  

Вывожу состояние кластера:
```bash
[elasticsearch@81184aca62b7 /]$ curl -X GET 'localhost:9200/_cluster/health?pretty'
{
  "cluster_name" : "netology_test",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 8,
  "active_shards" : 8,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 10,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 44.44444444444444
}
```

Удаляю все индексы и сразу же проверяю их список:
```bash
[elasticsearch@81184aca62b7 /]$ curl -X DELETE 'localhost:9200/_all'
{"acknowledged":true}[elasticsearch@81184aca62b7 /]$ 
[elasticsearch@81184aca62b7 /]$ curl -X GET 'localhost:9200/_cat/indices'
[elasticsearch@81184aca62b7 /]$ 
```

## Задача 3

В данном задании вы научитесь:
- создавать бэкапы данных
- восстанавливать индексы из бэкапов

Создайте директорию `{путь до корневой директории с elasticsearch в образе}/snapshots`.

Используя API [зарегистрируйте](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html#snapshots-register-repository) 
данную директорию как `snapshot repository` c именем `netology_backup`.

**Приведите в ответе** запрос API и результат вызова API для создания репозитория.

Создайте индекс `test` с 0 реплик и 1 шардом и **приведите в ответе** список индексов.

[Создайте `snapshot`](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html) 
состояния кластера `elasticsearch`.

**Приведите в ответе** список файлов в директории со `snapshot`ами.

Удалите индекс `test` и создайте индекс `test-2`. **Приведите в ответе** список индексов.

[Восстановите](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) состояние
кластера `elasticsearch` из `snapshot`, созданного ранее. 

**Приведите в ответе** запрос к API восстановления и итоговый список индексов.

Подсказки:
- возможно вам понадобится доработать `elasticsearch.yml` в части директивы `path.repo` и перезапустить `elasticsearch`  
<br>
<br>


Ответ:  
Под спойлеры спрячу промежуточные этапы выполнения ДЗ, их не просят приводить в ответах.  

Регистрирую директорию (она предварительно создана из Dockerfile):
```bash
[elasticsearch@81184aca62b7 /]$ curl -X PUT "localhost:9200/_snapshot/netology_backup?pretty" -H 'Content-Type: application/json' -d'
> {
>   "type": "fs",
>   "settings": {
>     "location": "/opt/elasticsearch-8.0.1/snapshots/"
>   }
> }
> '
{
  "acknowledged" : true
}
[elasticsearch@81184aca62b7 /]$ curl -X GET 'localhost:9200/_snapshot/netology_backup?pretty'
{
  "netology_backup" : {
    "type" : "fs",
    "settings" : {
      "location" : "/opt/elasticsearch-8.0.1/snapshots/"
    }
  }
}
```
<details>
  <summary>Console</summary>


Создаю индекс:
```bash
[elasticsearch@81184aca62b7 /]$ curl -X PUT "localhost:9200/test?pretty" -H 'Content-Type: application/json' -d'
> {
>   "settings": {
>     "index": {
>       "number_of_shards": 1,  
>       "number_of_replicas": 0 
>     }
>   }
> }
> '
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test"
}
```
</details>

Список индексов:
```bash
[elasticsearch@81184aca62b7 /]$ curl -X GET 'localhost:9200/_cat/indices'
green open test m4jiAlG1SlCll3a_e-7XdA 1 0 0 0 225b 225b
```
<details>
  <summary>Console</summary>

Создаю снапшот:
```bash
[elasticsearch@81184aca62b7 /]$ curl -X PUT "localhost:9200/_snapshot/netology_backup/my_best_snapshot?wait_for_completion=true&pretty"
{
  "snapshot" : {
    "snapshot" : "my_best_snapshot",
    "uuid" : "mtNH-bjdTe6cUT2iRQScPQ",
    "repository" : "netology_backup",
    "version_id" : 8000199,
    "version" : "8.0.1",
    "indices" : [
      ".geoip_databases",
      "test"
    ],
    "data_streams" : [ ],
    "include_global_state" : true,
    "state" : "SUCCESS",
    "start_time" : "2022-03-04T09:40:39.961Z",
    "start_time_in_millis" : 1646386839961,
    "end_time" : "2022-03-04T09:40:41.163Z",
    "end_time_in_millis" : 1646386841163,
    "duration_in_millis" : 1202,
    "failures" : [ ],
    "shards" : {
      "total" : 2,
      "failed" : 0,
      "successful" : 2
    },
    "feature_states" : [
      {
        "feature_name" : "geoip",
        "indices" : [
          ".geoip_databases"
        ]
      }
    ]
  }
}
```
</details>

Список файлов в директории со снапшотами:
```bash
[elasticsearch@81184aca62b7 /]$ ls -lh /opt/elasticsearch-8.0.1/snapshots/
total 36K
-rw-r--r-- 1 elasticsearch elasticsearch  849 Mar  4 09:40 index-0
-rw-r--r-- 1 elasticsearch elasticsearch    8 Mar  4 09:40 index.latest
drwxr-xr-x 4 elasticsearch elasticsearch 4.0K Mar  4 09:40 indices
-rw-r--r-- 1 elasticsearch elasticsearch  17K Mar  4 09:40 meta-mtNH-bjdTe6cUT2iRQScPQ.dat
-rw-r--r-- 1 elasticsearch elasticsearch  357 Mar  4 09:40 snap-mtNH-bjdTe6cUT2iRQScPQ.dat
```

<details>
  <summary>Console</summary>


Удаляю индекс `test`, создаю `test-2`:
```bash
[elasticsearch@81184aca62b7 /]$ curl -X DELETE "localhost:9200/test"
{"acknowledged":true}[elasticsearch@81184aca62b7 /]$ 
[elasticsearch@81184aca62b7 /]$ curl -X PUT "localhost:9200/test-2?pretty" -H 'Content-Type: application/json' -d'
> {
>   "settings": {
>     "index": {
>       "number_of_shards": 1,  
>       "number_of_replicas": 0 
>     }
>   }
> }
> '
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test-2"
}
```
</details>

Список индексов:
```bash
[elasticsearch@81184aca62b7 /]$ curl -X GET 'localhost:9200/_cat/indices?v'
health status index  uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test-2 lhChQdFsTkSqH54JDQrojQ   1   0          0            0       225b           225b
```

Восстанавливаю состояние кластера из бекапа:
```bash
[elasticsearch@81184aca62b7 /]$ curl -X POST "localhost:9200/_snapshot/netology_backup/my_best_snapshot/_restore?pretty"
{
  "accepted" : true
}
```

Итоговый список индексов:
```bash
[elasticsearch@81184aca62b7 /]$ curl -X GET 'localhost:9200/_cat/indices?v'
health status index  uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test-2 lhChQdFsTkSqH54JDQrojQ   1   0          0            0       225b           225b
green  open   test   p5U_LSWHQ2a-X5atiE8aJQ   1   0          0            0       225b           225b
```