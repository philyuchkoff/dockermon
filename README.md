dockermon
==========

Мониторинг Docker-хостов и контейнеров стеком из [Prometheus](https://prometheus.io/), [Grafana](http://grafana.org/), [cAdvisor](https://github.com/google/cadvisor), [NodeExporter](https://github.com/prometheus/node_exporter) и алертингом с [AlertManager](https://github.com/prometheus/alertmanager).

Это же решение с Docker Swarm: [stefanprodan/swarmprom](https://github.com/stefanprodan/swarmprom)

+ Интересный дашбоард для Prometheus Alertmanager: [prymitive/karma] (https://github.com/prymitive/karma)

***вставить картинку-схему связей компонентов***

# Установка

### Требования:

* Docker Engine >= 1.13
* Docker Compose >= 1.11

Скопировать репозиторий `dockermon` на докер-хост, перейти в директорию `dockermon` и запустить `compose up`:

    git clone https://github.com/philyuchkoff/dockermon && cd dockermon && ADMIN_USER=admin ADMIN_PASSWORD=admin docker-compose up -d

### Контейнеры:

* Prometheus http://<host-ip>:9090
* AlertManager http://<host-ip>:9093
* Grafana http://<host-ip>:3000
* NodeExporter (сборщик метрик хостов);
* cAdvisor (сборщик метрик контейнеров).
* Caddy (reverse proxy and basic auth provider for prometheus and alertmanager)

### Настройка Grafana

Перейдите на http://<host-ip>:3000 и авторизуйтесь c логином `admin` и паролем `changeme`. Пароль на свой можно изменить с помощью Grafana UI или изменив файл user.config
  
```
grafana:
  image: grafana/grafana:7.2.0
  env_file:
    - config

```
user.config должен выглядеть примерно так:
```
GF_SECURITY_ADMIN_USER=admin
GF_SECURITY_ADMIN_PASSWORD=changeme-заменить-на-нужный
GF_USERS_ALLOW_SIGN_UP=false
```
If you want to change the password, you have to remove this entry, otherwise the change will not take effect
```
- grafana_data:/var/lib/grafana
```

Из меню Grafana выберите пункт «Источники данных» (Data Sources) и кликните «Добавить источник данных» (Add Data Source). Чтобы добавить контейнеры Prometheus как источник данных, используйте следующие значения:


Имя: Prometheus
Тип: Prometheus
Url: http://prometheus:9090
Доступ: proxy

Теперь вы можете импортировать шаблоны панели управления из директории Grafana. Из меню Grafana выберите «Панель управления» и нажмите «Импорт».
