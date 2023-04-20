# Домашнее задание к занятию  «Очереди RabbitMQ» Андрей Дёмин


### Задание 1. Установка RabbitMQ

Используя Vagrant или VirtualBox, создайте виртуальную машину и установите RabbitMQ.
Добавьте management plug-in и зайдите в веб-интерфейс.

*Итогом выполнения домашнего задания будет приложенный скриншот веб-интерфейса RabbitMQ.*

![](img/1-1.png)
---
![](img/1-2.png)
---
![](img/1-3.png)


### Задание 2. Отправка и получение сообщений

Используя приложенные скрипты, проведите тестовую отправку и получение сообщения.
Для отправки сообщений необходимо запустить скрипт producer.py.

Для работы скриптов вам необходимо установить Python версии 3 и библиотеку Pika.
Также в скриптах нужно указать IP-адрес машины, на которой запущен RabbitMQ, заменив localhost на нужный IP.

```shell script
pip install pika
```

Зайдите в веб-интерфейс, найдите очередь под названием hello и сделайте скриншот.
После чего запустите второй скрипт consumer.py и сделайте скриншот результата выполнения скрипта

*В качестве решения домашнего задания приложите оба скриншота, сделанных на этапе выполнения.*

Для закрепления материала можете попробовать модифицировать скрипты, чтобы поменять название очереди и отправляемое сообщение.


![](img/2-1.png)
---
![](img/2-2.png)
---
![](img/2-3.png)
---
![](img/2-4.png)


### Задание 3. Подготовка HA кластера

Используя Vagrant или VirtualBox, создайте вторую виртуальную машину и установите RabbitMQ.
Добавьте в файл hosts название и IP-адрес каждой машины, чтобы машины могли видеть друг друга по имени.

Пример содержимого hosts файла:

```shell script
cat /etc/hosts
192.168.0.10 rmq01
192.168.0.11 rmq02
```
После этого ваши машины могут пинговаться по имени.

Затем объедините две машины в кластер и создайте политику ha-all на все очереди.

*В качестве решения домашнего задания приложите скриншоты из веб-интерфейса с информацией о доступных нодах в кластере и включённой политикой.*

Также приложите вывод команды с двух нод:

```shell script
rabbitmqctl cluster_status
```

Для закрепления материала снова запустите скрипт producer.py и приложите скриншот выполнения команды на каждой из нод:

```shell script
rabbitmqadmin get queue='hello'
```

После чего попробуйте отключить одну из нод, желательно ту, к которой подключались из скрипта, затем поправьте параметры подключения в скрипте consumer.py на вторую ноду и запустите его.

*Приложите скриншот результата работы второго скрипта.*

<ins>Ответ</ins>:

Вторая машина создана путем клонирования в VirtualBox c уже установленным  rabbitmq. На VM1 и VM2 для смены имени хоста выполняем команду:

```
hostnamectl set-hostname rabbit-node-1
```
```
hostnamectl set-hostname rabbit-node-2
```
![](img/3-1.png)

После этого добавляем ip и названия нод в файл /etc/hosts : 

![](img/3-2.png)

Далее на VM2 перезагружаем сервер rabbitmq и останавливаем службу для подключения к кластеру на базе VM1:

```
systemctl restart rabbitmq-server
```
```
rabbitmqctl stop_app
```
```
rabbitmqctl join_cluster rabbit@rabbit-node-1
```
![](img/3-3.png)

После подключения в кластер стартуем сервис на ноде 2:

```
rabbitmqctl start_app
```
Создаем политику ha-all:

```
rabbitmqctl set_policy ha-all "" '{"ha-mode":"all","ha-sync-mode":"automatic"}'
```

![](img/3-7.png)

Состояние кластера на VM1:

![](img/3-4.png)

Состояние кластера на VM2:

![](img/3-5.png)

![](img/3-6.png)
---
![](img/3-8.png)
---
![](img/3-9.png)
---
![](img/3-10.png)

Запуск скрипта producer.py и выполнения команды: 

```shell script
rabbitmqadmin get queue='hello'
```

на ноде 1:

![](img/3-11.png)

на ноде 2:

![](img/3-12.png)

Состояние нод кластера:

![](img/3-13.png)

Отключение первой ноды  и выполнение скрипта consumer.py на вторую ноду:

![](img/3-14.png)

Состояние нод кластера:

![](img/3-16.png)

Содержание скрипта consumer.py :

```
#!/usr/bin/env python
# coding=utf-8
import pika

credentials = pika.PlainCredentials('demin','password')
parameters = pika.ConnectionParameters('192.168.0.17',5672,'/',credentials)
connection = pika.BlockingConnection(parameters)
channel = connection.channel()
channel.queue_declare(queue='hello')

def callback(ch, method, properties, body):
    print(" [x] Received %r" % body)

channel.basic_consume (queue='hello', on_message_callback=callback, auto_ack=True)
channel.start_consuming()
```
![](img/3-15.png)


## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

### * Задание 4. Ansible playbook

Напишите плейбук, который будет производить установку RabbitMQ на любое количество нод и объединять их в кластер.
При этом будет автоматически создавать политику ha-all.

*Готовый плейбук разместите в своём репозитории.*

