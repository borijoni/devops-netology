# Домашнее задание к занятию "6.5. Elasticsearch"

## Задача 1

В этом задании вы потренируетесь в:
- установке elasticsearch
- первоначальном конфигурировании elastcisearch
- запуске elasticsearch в docker

Используя докер образ [elasticsearch:7](https://hub.docker.com/_/elasticsearch) как базовый:

- составьте Dockerfile-манифест для elasticsearch
- соберите docker-образ и сделайте `push` в ваш docker.io репозиторий
- запустите контейнер из получившегося образа и выполните запрос пути `/` c хост-машины

Требования к `elasticsearch.yml`:
- данные `path` должны сохраняться в `/var/lib` 
- имя ноды должно быть `netology_test`
```
cluster.name: netology_test
discovery.type: single-node
node.name: netology_test
path.data: /var/lib/data
path.logs: /var/lib/logs
path.repo: /var/lib/snapshots
network.host: 0.0.0.0
discovery.seed_hosts: ["127.0.0.1", "[::1]"]
```

В ответе приведите:
- текст Dockerfile манифеста
```
FROM elasticsearch:7.0.0
MAINTAINER Ivan Borimechkov <wabo@bk.ru>

COPY --chown=elasticsearch:elasticsearch elasticsearch.yml /usr/share/elasticsearch/config/

RUN mkdir /var/lib/logs \
    && chown elasticsearch:elasticsearch /var/lib/logs \
    && mkdir /var/lib/data \
    && chown elasticsearch:elasticsearch /var/lib/data \
    && mkdir /var/lib/snapshots \
    && chown elasticsearch:elasticsearch /var/lib/snapshots

USER elasticsearch

ENV PATH=$PATH:/usr/share/elasticsearch/bin

CMD ["elasticsearch"]

EXPOSE 9200 9300

```
- ссылку на образ в репозитории dockerhub
```
https://hub.docker.com/r/wabor/es
```
- ответ `elasticsearch` на запрос пути `/` в json виде
```
root@vagrant:/home/vagrant/ES# curl -X GET "http://127.0.0.1:9200/"
{
  "name" : "netology_test",
  "cluster_name" : "netology_test",
  "cluster_uuid" : "3WRxplBCT-Gjphbqt3DWog",
  "version" : {
    "number" : "7.0.0",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "b7e28a7",
    "build_date" : "2019-04-05T22:55:32.697037Z",
    "build_snapshot" : false,
    "lucene_version" : "8.0.0",
    "minimum_wire_compatibility_version" : "6.7.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

Подсказки:
- при сетевых проблемах внимательно изучите кластерные и сетевые настройки в elasticsearch.yml
- при некоторых проблемах вам поможет docker директива ulimit
- elasticsearch в логах обычно описывает проблему и пути ее решения
- обратите внимание на настройки безопасности такие как `xpack.security.enabled` 
- если докер образ не запускается и падает с ошибкой 137 в этом случае может помочь настройка `-e ES_HEAP_SIZE`
- при настройке `path` возможно потребуется настройка прав доступа на директорию

Далее мы будем работать с данным экземпляром elasticsearch.

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

```
curl -X PUT localhost:9200/ind-1 -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 1,  "number_of_replicas": 0 }}'
curl -X PUT localhost:9200/ind-2 -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 2,  "number_of_replicas": 1 }}'
curl -X PUT localhost:9200/ind-3 -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 4,  "number_of_replicas": 2 }}' 
```
Получите список индексов и их статусов, используя API и **приведите в ответе** на задание.
```
Получение списка индексов:

root@vagrant:/home/vagrant/ES# curl -X GET 'http://localhost:9200/_cat/indices?v'
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   ind-2 XGe6UMq9RlyQZAVjHaUW3g   2   1          0            0       460b           460b
green  open   ind-1 ViLH0H2ZQnC1st9e0z5q0w   1   0          0            0       230b           230b
yellow open   ind-3 QJ2wfKHsRS2wrqtXVTDMWQ   4   2          0            0       920b           920b

Получение статусов индексов:

root@vagrant:/home/vagrant/ES# curl -X GET 'http://localhost:9200/_cluster/health/ind-1?pretty'
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
root@vagrant:/home/vagrant/ES# curl -X GET 'http://localhost:9200/_cluster/health/ind-2?pretty'
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
  "active_shards_percent_as_number" : 41.17647058823529
}
root@vagrant:/home/vagrant/ES# curl -X GET 'http://localhost:9200/_cluster/health/ind-3?pretty'
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
  "active_shards_percent_as_number" : 41.17647058823529
}
```

Получите состояние кластера `elasticsearch`, используя API.
```
root@vagrant:/home/vagrant/ES# curl -XGET localhost:9200/_cluster/health/?pretty=true
{
  "cluster_name" : "netology_test",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 7,
  "active_shards" : 7,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 10,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 41.17647058823529
}
```
Как вы думаете, почему часть индексов и кластер находится в состоянии yellow?
```
Сосотяние yellow говорит о том что индексы не реплецируются, хотя явно указаны.
А не реплецируются из за отсутсвия серверов для реплецирования.
```
Удалите все индексы.
```
root@vagrant:/home/vagrant/ES# curl -X DELETE 'http://localhost:9200/ind-1?pretty'
{
  "acknowledged" : true
}
root@vagrant:/home/vagrant/ES# curl -X DELETE 'http://localhost:9200/ind-2?pretty'
{
  "acknowledged" : true
}
root@vagrant:/home/vagrant/ES# curl -X DELETE 'http://localhost:9200/ind-3?pretty'
{
  "acknowledged" : true
}
```

**Важно**

При проектировании кластера elasticsearch нужно корректно рассчитывать количество реплик и шард,
иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.

## Задача 3

В данном задании вы научитесь:
- создавать бэкапы данных
- восстанавливать индексы из бэкапов

Создайте директорию `{путь до корневой директории с elasticsearch в образе}/snapshots`.

Используя API [зарегистрируйте](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html#snapshots-register-repository) 
данную директорию как `snapshot repository` c именем `netology_backup`.

**Приведите в ответе** запрос API и результат вызова API для создания репозитория.
```
root@vagrant:/home/vagrant/ES# curl -XPOST localhost:9200/_snapshot/netology_backup?pretty -H 'Content-Type: application/json' -d'{"type": "fs", "settings": { "location":"/var/lib/snapshots" }}'
{
  "acknowledged" : true
}
root@vagrant:/home/vagrant/ES# curl http://localhost:9200/_snapshot/netology_backup?pretty
{
  "netology_backup" : {
    "type" : "fs",
    "settings" : {
      "location" : "/var/lib/snapshots"
    }
  }
}
```
Создайте индекс `test` с 0 реплик и 1 шардом и **приведите в ответе** список индексов.
```
root@vagrant:/home/vagrant/ES# curl -X PUT localhost:9200/test -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 1,  "number_of_replicas": 0 }}'

Список индексов:

root@vagrant:/home/vagrant/ES# curl -X GET 'http://localhost:9200/_cat/indices?v'
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test  DtI_AJroRHyGY_AL2EtcMQ   1   0          0            0       283b           283b

```
[Создайте `snapshot`](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html) 
состояния кластера `elasticsearch`.
```
root@vagrant:/home/vagrant/ES# curl -X PUT localhost:9200/_snapshot/netology_backup/elasticsearch?wait_for_completion=true
{"snapshot":{"snapshot":"elasticsearch","uuid":"fvtM0eRGSlG3UC4NnUAW3Q","version_id":7000099,"version":"7.0.0","indices":["test"],"include_global_state":true,"state":"SUCCESS","start_time":"2022-07-27T14:02:39.349Z","start_time_in_millis":1658930559349,"end_time":"2022-07-27T14:02:39.406Z","end_time_in_millis":1658930559406,"duration_in_millis":57,"failures":[],"shards":{"total":1,"failed":0,"successful":1}}}
```

**Приведите в ответе** список файлов в директории со `snapshot`ами.
```
root@vagrant:/home/vagrant/ES# docker exec -it es bash
[elasticsearch@a17b86500d33 ~]$ pwd
/usr/share/elasticsearch
[elasticsearch@a17b86500d33 ~]$ cd /var/lib/snapshots/
[elasticsearch@a17b86500d33 snapshots]$ pwd
/var/lib/snapshots
[elasticsearch@a17b86500d33 snapshots]$ ls -la
total 52
drwxr-xr-x 1 elasticsearch elasticsearch  4096 Jul 27 14:02 .
drwxr-xr-x 1 root          root           4096 Jul 27 12:26 ..
-rw-rw-r-- 1 elasticsearch elasticsearch   172 Jul 27 14:02 index-0
-rw-rw-r-- 1 elasticsearch elasticsearch     8 Jul 27 14:02 index.latest
drwxrwxr-x 3 elasticsearch elasticsearch  4096 Jul 27 14:02 indices
-rw-rw-r-- 1 elasticsearch elasticsearch 21209 Jul 27 14:02 meta-fvtM0eRGSlG3UC4NnUAW3Q.dat
-rw-rw-r-- 1 elasticsearch elasticsearch   244 Jul 27 14:02 snap-fvtM0eRGSlG3UC4NnUAW3Q.dat
```
Удалите индекс `test` и создайте индекс `test-2`. **Приведите в ответе** список индексов.
```
root@vagrant:/home/vagrant/ES# curl -X DELETE 'http://localhost:9200/test?pretty'
{
  "acknowledged" : true
}
root@vagrant:/home/vagrant/ES# curl -X PUT localhost:9200/test-2?pretty -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 1,  "number_of_replicas": 0 }}'
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test-2"
}
root@vagrant:/home/vagrant/ES# curl -X GET 'http://localhost:9200/_cat/indices?v'
health status index  uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test-2 QeATZt-kS66Wm3NRzqbNPg   1   0          0            0       230b           230b
```

[Восстановите](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) состояние
кластера `elasticsearch` из `snapshot`, созданного ранее. 

**Приведите в ответе** запрос к API восстановления и итоговый список индексов.
```
root@vagrant:/home/vagrant/ES# curl -X POST localhost:9200/_snapshot/netology_backup/elasticsearch/_restore?pretty -H 'Content-Type: application/json' -d'{"include_global_state":true}'
{
  "accepted" : true
}
root@vagrant:/home/vagrant/ES# curl -X GET 'http://localhost:9200/_cat/indices?v'
health status index  uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test   nQ24bjOgTRSePp6qvc6ORg   1   0          0            0       283b           283b
green  open   test-2 QeATZt-kS66Wm3NRzqbNPg   1   0          0            0       230b           230b
```
Подсказки:
- возможно вам понадобится доработать `elasticsearch.yml` в части директивы `path.repo` и перезапустить `elasticsearch`

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
