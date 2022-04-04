### Как сдавать задания

Вы уже изучили блок «Системы управления версиями», и начиная с этого занятия все ваши работы будут приниматься ссылками на .md-файлы, размещённые в вашем публичном репозитории.

Скопируйте в свой .md-файл содержимое этого файла; исходники можно посмотреть [здесь](https://raw.githubusercontent.com/netology-code/sysadm-homeworks/devsys10/04-script-03-yaml/README.md). Заполните недостающие части документа решением задач (заменяйте `???`, ОСТАЛЬНОЕ В ШАБЛОНЕ НЕ ТРОГАЙТЕ чтобы не сломать форматирование текста, подсветку синтаксиса и прочее, иначе можно отправиться на доработку) и отправляйте на проверку. Вместо логов можно вставить скриншоты по желани.

# Домашнее задание к занятию "4.3. Языки разметки JSON и YAML"


## Обязательная задача 1
Мы выгрузили JSON, который получили через API запрос к нашему сервису:
```
    { "info" : "Sample JSON output from our service\t",
        "elements" :[
            { "name" : "first",
            "type" : "server",
            "ip" : 7175 
            }
            { "name" : "second",
            "type" : "proxy",
            "ip : 71.78.22.43
            }
        ]
    }
```
  Нужно найти и исправить все ошибки, которые допускает наш сервис

```
    { "info" : "Sample JSON output from our service\t",
        "elements" :
        [
            { "name" : "first",
            "type" : "server",
            "ip" : "7175"
            },
            { "name" : "second",
            "type" : "proxy",
            "ip" : "71.78.22.43"
            }
        ]
    }
```

## Обязательная задача 2
В прошлый рабочий день мы создавали скрипт, позволяющий опрашивать веб-сервисы и получать их IP. К уже реализованному функционалу нам нужно добавить возможность записи JSON и YAML файлов, описывающих наши сервисы. Формат записи JSON по одному сервису: `{ "имя сервиса" : "его IP"}`. Формат записи YAML по одному сервису: `- имя сервиса: его IP`. Если в момент исполнения скрипта меняется IP у сервиса - он должен так же поменяться в yml и json файле.

### Ваш скрипт:
```python

#!/usr/bin/env python3

import os
import socket
import json
import yaml

host_json="hosts.json"
host_yaml="host.yaml"
servers={'drive.google.com': '' ,'mail.google.com' : '' ,'google.com': ''}

for s in servers:
    servers[s]=socket.gethostbyname(s)

if os.path.exists(host_json)==False:
    with open(host_json, 'w') as js:
        json.dump(servers, js, indent=2)

if os.path.exists(host_yaml)==False:
    with open(host_yaml, 'w') as ym:
        yaml.dump(servers, ym, indent=4)

with open(host_json) as js:
    js_dict=json.load(js)
    for line in js_dict:
        if js_dict[line]==socket.gethostbyname(line):
            print(line+" "+servers[line])
        else:
            print("ERROR "+line+" IP mismatch: "+js_dict[line]+" "+socket.gethostbyname(line))

with open(host_json, 'w') as js:
    json.dump(servers, js, indent=2)

with open(host_yaml, 'w') as ym:
    yaml.dump(servers, ym, indent=4)


```

### Вывод скрипта при запуске при тестировании:
```
drive.google.com 172.217.18.110
mail.google.com 142.250.185.69
ERROR google.com IP mismatch:142.20.184.238 142.250.184.238
```

### json-файл(ы), который(е) записал ваш скрипт:
```json
{
  "drive.google.com": "172.217.18.110",
  "mail.google.com": "142.250.185.69",
  "google.com": "142.250.184.238"
}


```

### yml-файл(ы), который(е) записал ваш скрипт:
```yaml
drive.google.com: 172.217.18.110
google.com: 142.250.184.238
mail.google.com: 142.250.185.69

```
