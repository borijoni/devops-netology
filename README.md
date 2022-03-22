# Домашнее задание к занятию "4.2. Использование Python для решения типовых DevOps задач"

## Обязательная задача 1

Есть скрипт:
```python
#!/usr/bin/env python3
a = 1
b = '2'
c = a + b
```

### Вопросы:
| Вопрос  | Ответ |
| ------------- | ------------- |
| Какое значение будет присвоено переменной `c`?  | Traceback (most recent call last): File "<stdin>", line 1, in <module> TypeError: unsupported operand type(s) for +: 'int' and 'str'  |
| Как получить для переменной `c` значение 12?  | c=str(a)+b  |
| Как получить для переменной `c` значение 3?  | c=a+int(b)  |

## Обязательная задача 2
Мы устроились на работу в компанию, где раньше уже был DevOps Engineer. Он написал скрипт, позволяющий узнать, какие файлы модифицированы в репозитории, относительно локальных изменений. Этим скриптом недовольно начальство, потому что в его выводе есть не все изменённые файлы, а также непонятен полный путь к директории, где они находятся. Как можно доработать скрипт ниже, чтобы он исполнял требования вашего руководителя?

```python
#!/usr/bin/env python3

import os

bash_command = ["cd ~/netology/sysadm-homeworks", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
is_change = False
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(prepare_result)
        break
```

### Ваш скрипт:
```python
#!/usr/bin/env python3

import os

bash_command = ["cd ~/netology/sysadm-homeworks", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
#is_change = False
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(str(bash_command[0])[3:]+"/"+prepare_result)
#        break

```

### Вывод скрипта при запуске при тестировании:
```
root@test:~# ./test_git.py
~/netology/sysadm-homeworks/README.md
~/netology/sysadm-homeworks/bt/bats.txt
~/netology/sysadm-homeworks/has_been_moved.txt

```

## Обязательная задача 3
1. Доработать скрипт выше так, чтобы он мог проверять не только локальный репозиторий в текущей директории, а также умел воспринимать путь к репозиторию, который мы передаём как входной параметр. Мы точно знаем, что начальство коварное и будет проверять работу этого скрипта в директориях, которые не являются локальными репозиториями.

### Ваш скрипт:
```python
#!/usr/bin/env python3

import os
import sys

path_rep=sys.argv[1]
bash_command = ["cd "+path_rep, "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
#is_change = False
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(str(bash_command[0])[3:]+"/"+prepare_result)
#        break


```

### Вывод скрипта при запуске при тестировании:
```
root@test:~# ./test_git3.py /home/admin-msk/git/devops-netology
/home/admin-msk/git/devops-netology/README.md
/home/admin-msk/git/devops-netology/has_been_moved.txt

```

## Обязательная задача 4
1. Наша команда разрабатывает несколько веб-сервисов, доступных по http. Мы точно знаем, что на их стенде нет никакой балансировки, кластеризации, за DNS прячется конкретный IP сервера, где установлен сервис. Проблема в том, что отдел, занимающийся нашей инфраструктурой очень часто меняет нам сервера, поэтому IP меняются примерно раз в неделю, при этом сервисы сохраняют за собой DNS имена. Это бы совсем никого не беспокоило, если бы несколько раз сервера не уезжали в такой сегмент сети нашей компании, который недоступен для разработчиков. Мы хотим написать скрипт, который опрашивает веб-сервисы, получает их IP, выводит информацию в стандартный вывод в виде: <URL сервиса> - <его IP>. Также, должна быть реализована возможность проверки текущего IP сервиса c его IP из предыдущей проверки. Если проверка будет провалена - оповестить об этом в стандартный вывод сообщением: [ERROR] <URL сервиса> IP mismatch: <старый IP> <Новый IP>. Будем считать, что наша разработка реализовала сервисы: `drive.google.com`, `mail.google.com`, `google.com`.

### Ваш скрипт:
```python
#!/usr/bin/env python3

import os
import socket

host_file="hosts.txt"
servers=["drive.google.com","mail.google.com","google.com"]

if os.path.exists(host_file)==False:
    with open(host_file, 'w') as f:
        for s in servers:
            f.write(s+" "+socket.gethostbyname(s)+"\n")
            print(s+" "+socket.gethostbyname(s))
else:
    with open(host_file) as f:
        for line in f:
            if line.strip()==line.split(' ')[0]+" "+socket.gethostbyname(line.split(' ')[0]):
                print(line.strip())
            else:
                old_string=line.strip()
                domain=line.split(' ')[0]
                old_ip=old_string.replace(domain, ' ')
                print("ERROR "+line.split(' ')[0]+" IP mismatch:"+old_ip+" "+socket.gethostbyname(line.split(' ')[0]))

```

### Вывод скрипта при запуске при тестировании:
```
root@test:~#./test_hosts.py
drive.google.com 142.250.74.206
mail.google.com 142.250.185.69
google.com 172.217.16.142

Вносим изменения в файл hosts.txt

root@test:~#./test_hosts.py
drive.google.com 142.250.74.206
mail.google.com 142.250.185.69
ERROR google.com IP mismatch:  172.217.16.14 172.217.16.142

```

