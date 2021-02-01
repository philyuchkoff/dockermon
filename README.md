dockermon
==========

Мониторинг Docker-хостов и контейнеров стеком из [Prometheus](https://prometheus.io/), [Grafana](http://grafana.org/), [cAdvisor](https://github.com/google/cadvisor), [NodeExporter](https://github.com/prometheus/node_exporter) и алертингом с [AlertManager](https://github.com/prometheus/alertmanager).

:exclamation: Это же решение с Docker Swarm: [stefanprodan/swarmprom](https://github.com/stefanprodan/swarmprom)

:+1: Интересный дашбоард для Prometheus Alertmanager: [Karma](https://github.com/prymitive/karma)

***вставить картинку-схему связей компонентов***

## Установка

### Требования:

* Docker Engine >= 1.13
* Docker Compose >= 1.11

Скопировать репозиторий `dockermon` на докер-хост, перейти в директорию `dockermon` и запустить `compose up`:

    git clone https://github.com/philyuchkoff/dockermon
    cd dockermon
    ADMIN_USER=admin ADMIN_PASSWORD=admin docker-compose up -d

### Будут запущены контейнеры:

* Prometheus `http://<host-ip>:9090`
* AlertManager `http://<host-ip>:9093`
* Grafana `http://<host-ip>:3000`
* NodeExporter (сборщик метрик хостов);
* cAdvisor (сборщик метрик контейнеров).
* Caddy (reverse proxy and basic auth provider for prometheus and alertmanager)

## Настройка Grafana

Перейдите на http://<host-ip>:3000 и авторизуйтесь c логином `admin` и паролем `admin`. После первого входа будет предложено изменить пароль на новый. 
Учетные данные можно изменить в `docker-compose.yml` или указав переменные среды `ADMIN_USER` и `ADMIN_PASSWORD` при запуске, примерно так:

`ADMIN_USER=user ADMIN_PASSWORD=password docker-compose up -d`

Или учетные данные можно добавить непосредственно в конфиг Grafana:  
```
grafana:
  image: grafana/grafana:7.2.0
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

Gafana поддерживает аутентификацию, а Prometheus и AlertManager нет. Если нужно - удалите отображение портов Prometheus и AlertManager из docker-compose и используйте NGINX как реверс-прокси.

В Grafana предварительно уже настроены дашборды и в качестве default data source указан Prometheus. Из меню Grafana выберите Data Sources - Add Data Source и укажите контейнеры Prometheus как источник данных:

- Name: Prometheus
- Type: Prometheus
- Url: `http://prometheus:9090`
- Access: Server (default)

### Docker Host Dashboard

### Docker Containers Dashboard

### Monitor Services Dashboard

## Масштабирование

Для контроля большего количества хостов нужно запустить ***node-exporter*** и контейнер ***cAdvisor*** на каждом новом хосте и указать эти хосты в конфиге Prometheus для скрапинга.


qqqqqqqqqqqqqqqqqqqqqq
Теперь вы можете импортировать шаблоны панели управления из директории Grafana. Из меню Grafana выберите «Панель управления» и нажмите «Импорт».


https://habr.com/ru/company/southbridge/blog/314212/

https://github.com/stefanprodan/dockprom

https://stefanprodan.com/2016/a-monitoring-solution-for-docker-hosts-containers-and-containerized-services/
