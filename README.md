Система мониторинга Docker-хостов и контейнеров стеком из [Prometheus](https://prometheus.io/), [Grafana](http://grafana.org/), [cAdvisor](https://github.com/google/cadvisor), [NodeExporter](https://github.com/prometheus/node_exporter) и алертингом с [AlertManager](https://github.com/prometheus/alertmanager).

За основу взята идея Stefan Prodan [dockprom](https://github.com/philyuchkoff/dockprom).

:exclamation: Это же решение с Docker Swarm: [stefanprodan/swarmprom](https://github.com/stefanprodan/swarmprom)


### Общая схема
![Схема](https://github.com/philyuchkoff/dockermon/blob/main/screenshots/dockermon.png)

## Установка

### Требования:

- Docker Engine >= 1.13
- Docker Compose >= 1.11

Скопировать репозиторий `dockermon` на докер-хост, перейти в директорию `dockermon` и запустить `compose up`:

    git clone https://github.com/philyuchkoff/dockermon
    cd dockermon
    ADMIN_USER=admin ADMIN_PASSWORD=admin docker-compose up -d

В результате будут запущены контейнеры:

- Prometheus `http://<host-ip>:9090`
- AlertManager `http://<host-ip>:9093`
- Grafana `http://<host-ip>:3000`
- NodeExporter (сборщик метрик хостов);
- cAdvisor (сборщик метрик контейнеров).
- Caddy (reverse proxy and basic auth provider for prometheus and alertmanager)

## Настройка Grafana

Перейдите на `http://<host-ip>:3000` и авторизуйтесь c логином `admin` и паролем `admin`. После первого входа будет предложено изменить пароль на новый. 
Учетные данные можно изменить в `docker-compose.yml` или указав переменные среды `ADMIN_USER` и `ADMIN_PASSWORD` при запуске, примерно так:

`ADMIN_USER=user ADMIN_PASSWORD=password docker-compose up -d`

Или учетные данные можно добавить непосредственно в конфиг Grafana:  
```
grafana:
  image: grafana/grafana:7.4.0
  env_file:
    - config
```
и в этом случае у файла `config` должен быть вот такой формат:
```
GF_SECURITY_ADMIN_USER=user
GF_SECURITY_ADMIN_PASSWORD=password
GF_USERS_ALLOW_SIGN_UP=false
```

Если вы решите поменять пароль, то нужно удалить вот эту строку:
```
- grafana_data:/var/lib/grafana
```
если ее не удалить - изменения не применятся, пароль не поменяется.

:exclamation: Gafana поддерживает аутентификацию, а Prometheus и AlertManager нет, то есть, доступ к Prometheus и AlertManager открыт для всех. Если это не то, что нужно - удалите expose портов Prometheus и AlertManager из `docker-compose.yml` и используйте реверс-прокси (Nginx или Caddy, например).

В Grafana предварительно уже настроены дашборды и в качестве default data source указан Prometheus. Из меню Grafana выберите `Data Sources` - `Add Data Source` и укажите контейнеры Prometheus как источник данных:

- Name: Prometheus
- Type: Prometheus
- Url: `http://prometheus:9090`
- Access: Server (default)

### Docker Host Dashboard

![Схема](https://github.com/philyuchkoff/dockermon/blob/main/screenshots/dockerhost.jpg)

В этом дашбоарде для графика `Free Storage` необходимо будет поправить тип файловой системы, заменив мою `ext4` на ту, которую используете вы. Меняется в `grafana/provisioning/dashboards/docker_host.json` в строке 521:
````
"expr": "сумма (node_filesystem_free_bytes {fstype = \" ext4 \ "})",
````
Правильное значение для своей системы можно посмотреть, сделав запрос `node_filesystem_free_bytes` в Prometheus.

### Docker Containers Dashboard

![Схема](https://github.com/philyuchkoff/dockermon/blob/main/screenshots/dockercontainers.jpg)

### Monitor Services Dashboard

### Node Exporter Dashboard

## Alerts
В файле [alert.rules](https://github.com/philyuchkoff/dockermon/blob/main/prometheus/alert.rules) определены алармы для групп:
- Host (Docker Host)
- Prometheus (сам Prometheus)
- Containers (Docker Containers)
- Targets

Чтобы изменить/удалить/добавить правила, нужно поправить правила в указанном файле и заставить Prometheus перечитать этот файл, выполнив HTTP POST запрос в Prometheus:

    curl -X POST http://LOGIN:PASSWORD@<host-ip>:9090/-/reload
    
Работает, только если Prometheus запущен с флагом `--web.enable-lifecycle`

### Настройка
Веб-интерфейс AlertManager доступен по адресу `http://<host-ip>:9093`
AlertManager отвечает за обработку алармов, которые отправляет Prometheus, и может отправлять их в разные системы, полный список которых можно посмотреть в [документации](https://prometheus.io/docs/alerting/latest/configuration/)
Отправку нотификаций в мессенджеры, почту и т.п. можно настроить в файле [alertmanager/config.yml](https://github.com/philyuchkoff/dockermon/blob/main/alertmanager/config.yml)

## Масштабирование

Если нужно подключить еще какие-то хосты, необходимо установить и запустить ***node-exporter*** и контейнер ***cAdvisor*** на каждом новом хосте и указать эти хосты в конфиге Prometheus.

В Prometheus не стоит ставить retention period большим. Если нужно хранить метрики долго - ставьте [VictoriaMetrics](https://victoriametrics.com/) (или Cortex, Thanos, M3DB...) и храните метрики там. Мне нравится VictoriaMetrics, single-node version хватит с избытком, но, если вам не хватит - у нее есть cluster version.

Для высокодоступности и отказоустойчивости Prometheus просто поставьте два идентичных Prometheus c одинаковым конфигурационным файлом (`prometheus.yml`) на двух серверах, чтобы они собирали одни и те же метрики. При этом сервер A будет дополнительно мониторить сервер B и наоборот.

Alertmanager может работать в кластерной конфигурации, умеет дедуплицировать данные с разных серверов Prometheus и может связываться с другими копиями Alertmanager, чтобы не отправлять несколько одинаковых оповещений. Ставьте Alertmanager на каждый сервер Prometheus.

# Что нового
- 11.02.2021 Версия Grafana изменена на 7.4.0
## Планы
- :+1: добавить интересный дашбоард для Prometheus Alertmanager: [Karma](https://github.com/prymitive/karma)
- нотификации от AlertManager в Telegram
- добавить [бэкапилку Prometheus TSDB](https://github.com/philyuchkoff/prometheus-tsdb-dump) (она же импортер метрик из Prometheus в VictoriaMetrics)
- добавить механизм бэкапа/восстановление Grafana, пока [скриптами](https://github.com/philyuchkoff/grafana-backup)
- ...

