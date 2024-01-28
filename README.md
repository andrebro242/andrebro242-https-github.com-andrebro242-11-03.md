Домашнее задание по занятости «ELK» Брюхов А SYS-26

Задание 1. Elasticsearch
Установите и запустите Elasticsearch, после чего поменяйте параметр имя_кластера на случайный.

Решение 1

1. Установка Elasticsearch:

sudo apt update
sudo apt install openjdk-11-jre
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo sh -c 'echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" > /etc/apt/sources.list.d/elastic-7.x.list'
sudo apt update
sudo apt install elasticsearch

2.Запуск Elasticsearch:

sudo service elasticsearch start

в файле /etc/elasticsearch/elasticsearch.yml

чтобы elasticsearch слушал все сетевые интерфейсы, настроил параметр:

network.host: 0.0.0.0

# systemctl restart elasticsearch.service

Смотрим, что получилось:

# netstat -tulnp | grep 9200
tcp6       0      0 127.0.0.1:9200          :::*                    LISTEN      19788/java

3.Изменение имени кластера:

Редактирую файл конфигурации Elasticsearch:

sudo nano /etc/elasticsearch/elasticsearch.yml

Внутри файла меняю параметр cluster.name:

cluster.name: netology-logging

4.Перезапуск Elasticsearch:

sudo service elasticsearch restart

5.Проверка состояния кластера:

Выполняю команду curl для получения состояния кластера Elasticsearch:

    curl -X GET 'localhost:9200/_cluster/health?pretty'

![Задание 1](Решение%1.png)

Задание 2. Кибана
Установите и запустите Кибану.

Решение 2

1.Установка Kibana:

sudo apt update
sudo apt install kibana

2.Запуск Kibana:

sudo service kibana start

# systemctl status kibana.service

# netstat -tulnp | grep 5601
tcp        0      0 127.0.0.1:5601          0.0.0.0:*               LISTEN      1487/node

3.Проверка файла конфигурации Kibana:

Редактирую:

sudo nano /etc/kibana/kibana.yml

Строка:

server.host: "0.0.0.0"
elasticsearch.hosts: ["http://localhost:9200"]

4.Перезапустил Kibana:

sudo service kibana restart

5.Проверка интерфейса Kibana:

http://127.0.0.1:5601/app/dev_tools#/console

В консоли выполнил запрос:

    GET /_cluster/health?pretty

![Задание 2](Решение%2.png)

Задание 3. Logstash
Установите и запустите Logstash и Nginx. С помощью Logstash отправьте лог доступа Nginx в Elasticsearch.

Решение 3

1.Установка Nginx

2.Установка Logstash:

sudo apt update
sudo apt install logstash

3.Настройка Logstash для Nginx:

Создал файл конфигурации Logstash /etc/logstash/conf.d/nginx.conf:

input {
  # Конфигурация ввода для логов Nginx
  file {
    path => "/var/log/nginx/access.log"
    start_position => "beginning"
  }
}

filter {
    grok {
      match => { "message" => "%{IPORHOST:remote_ip} - %{DATA:user_name}
\[%{HTTPDATE:access_time}\] \"%{WORD:http_method} %{DATA:url}
HTTP/%{NUMBER:http_version}\" %{NUMBER:response_code} %{NUMBER:body_sent_bytes}
\"%{DATA:referrer}\" \"%{DATA:agent}\"" }
    }
    mutate {
        remove_field => [ "host" ]
    }
}

output {
  # Конфигурация вывода для отправки данных в Elasticsearch
  elasticsearch {
    hosts => ["127.0.0.1:9200"]
    index => "nginx_logs"
  }
}


4.Запуск Logstash:

sudo service logstash start

5.Проверяю интерфейс Kibana:

    http://192.168.6.128:5601

![Задание 3](Решение%3.png)

Задание 4. Filebeat.
Установите и запустите Filebeat. Переключите поставку журналов Nginx с Logstash на Filebeat.

Решение 4

1.Установка Filebeat:

sudo apt update
sudo apt install filebeat

2.Настройка Filebeat для Nginx:
Редактирую файл конфигурации Filebeat /etc/filebeat/filebeat.yml:

filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log

output.elasticsearch:
  hosts: ["localhost:9200"]

3.Запуск Filebeat:

sudo service filebeat start

Проверяю интерфейс Kibana:

http://127.0.0.1:5601

![Задание 4](Решение%4.png)
