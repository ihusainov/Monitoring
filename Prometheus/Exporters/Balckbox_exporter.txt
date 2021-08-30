Установка
Для мониторинга серверов по протоколу ICMP нужно использовать https://github.com/prometheus/blackbox_exporter. 
Скачать и распаковать последнюю версию с страницы release на сервер мониторинга. Создать конфигурационные файлы и файл сервиса

**/etc/systemd/system/blackbox.service**
```bash
[Unit]
Description=BlackBox Exporter
Wants=network-online.target
After=network-online.target
 
[Service]
User=prometheus
Group=prometheus
Type=simple
CapabilityBoundingSet=CAP_NET_RAW CAP_NET_ADMIN
EnvironmentFile=-/etc/sysconfig/blackbox
ExecStart=/usr/local/sbin/blackbox_exporter $OPTIONS
 
[Install]
WantedBy=multi-user.target
```

**/etc/sysconfig/blackbox**
```bash
OPTIONS='--config.file="/etc/blackbox/blackbox.yml" --web.listen-address="127.0.0.1:9115"'
```

**/etc/blackbox/blackbox.yml**
```bash
modules:
  http_2xx:
    prober: http
  http_post_2xx:
    prober: http
    http:
      method: POST
  tcp_connect:
    prober: tcp
  pop3s_banner:
    prober: tcp
    tcp:
      query_response:
      - expect: "^+OK"
      tls: true
      tls_config:
        insecure_skip_verify: false
  ssh_banner:
    prober: tcp
    tcp:
      query_response:
      - expect: "^SSH-2.0-"
  irc_banner:
    prober: tcp
    tcp:
      query_response:
      - send: "NICK prober"
      - send: "USER prober prober prober :prober"
      - expect: "PING :([^ ]+)"
        send: "PONG ${1}"
      - expect: "^:[^ ]+ 001"
  icmp:
    prober: icmp
    timeout: 5s
    icmp:
      preferred_ip_protocol: "ip4"
 
# vim: ts=2 sw=2 autoindent
```

setcap cap_net_raw=+ep /usr/local/sbin/blackbox_exporter

**Мониторинг серверов по ICMP**

**/etc/prometheus/prometheus.yml**
```bash
- job_name: 'icmp_ping'
  scrape_timeout: 5s
  metrics_path: /probe
  params:
    module: [icmp]
  file_sd_configs:
    - files:
      - /etc/prometheus/scrapers/node_exporter/*.yml
      - /etc/prometheus/scrapers/node_exporter/*.yaml
  relabel_configs:
    - source_labels:  [__address__]
      regex:  '([^:]*)(:\d+)?'
      replacement:  '$1'
      target_label: __param_target
    - source_labels:  [__param_target]
      target_label: instance
    - target_label: __address__
      replacement:  127.0.0.1:9115
```

**Мониторинг ресурсов по http**
 
Требуется установленный и настроенный Blackbox_exporter
Для мониторинга ресурсов по протоколу http можно воспользоваться уже установленным blackbox.
```bash
- job_name: 'http_2xx'
  scrape_timeout: 15s
  metrics_path: /probe
  params:
    module: [http_2xx]
  file_sd_configs:
    - files:
      - /etc/prometheus/scrapers/http_2xx/*.yml
      - /etc/prometheus/scrapers/http_2xx/*.yaml
  relabel_configs:
    - source_labels:  [__address__]
      target_label: __param_target
    - source_labels:  [__param_target]
      target_label: instance
    - target_label: __address__
      replacement:  127.0.0.1:9115
```

**Файл содержащий цели для мониторинга имеет такой шаблон**
```bash
- targets:
  - ""
  labels:
    "site": ""
    "project": ""
    "category": ""
    "dc": ""
Где:

в targets указывается адрес который будет опрашиваться
site - имя опрашиваемого ресурса, иногда сайты имеют специальные url для мониторинга, но вот отображать такие url в grafana не в совсем удобно, поэтому добавляется метка site, которая содержит более удобочитаемое значение. Например: target - https://site.domain.com/api/health?key=securekey, тогда site удобно указать как https://site.domain.com
project - имя проекта к которому относится наблюдаемый ресурс
category - категория к которому относится наблюдаемый ресурс: customer, internal,public и тд
dc - датацентр в котором находится наблюдаемый ресурс
конечно добавить метки можно и другие 
```

**Пример алертов, недоступность ресурса по http**
```bash
- alert: "Http resource not available"
  expr: probe_success{job="http_2xx"} == 0
  for: 5m
  labels:
    severity: critical
    priority: P1
  annotations:
    summary: "Http resource {{ $labels.site }} not available"
    description: "Ресурс {{ $labels.site }} не доступен по http более 5 минут"

#Проблемы в датацентре
- alert: "All http resources in dc not available"
  expr: count(probe_success{job="http_2xx"}) by (dc) - count(probe_success{job="http_2xx"}==0) by (dc) == 0
  for: 5m
  labels:
    severity: critical
    priority: P1
  annotations:
    summary: "All http resources in datacenter {{ $labels.dc }} not available"
    description: "Все http ресурсы в датацентре {{ $labels.dc }} не доступны более 5 минут, возможно проблемы в датацентре"
```
