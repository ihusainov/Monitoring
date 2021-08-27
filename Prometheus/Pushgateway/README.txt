Общее описание
https://github.com/prometheus/pushgateway

Документация по установке и настройке
https://github.com/prometheus/client_ruby#pushgateway

Плагины для различных языков программирования
https://prometheus.io/docs/instrumenting/pushing/


Настройка prometheus на получение данных из Pushgateway

Файл prom.yml

global: null
scrape_interval: 5s
scrape_timeout: 2s
evaluation_interval: 15s

scrape_configs:
 - job_name: pushgateway
   honor_labels: true
   static_configs:
     - targets:
     - 'pushgateway:9091'

Где honor_lables, разрешает конфликты имён лэйблов, то есть например,
если у вашего сервиса есть метрика с лэблом "X" и у pushgateway, есть лэйбл "X", то при honor_lables=false у вас будет лэйбл "X" с pushgateway и "exported_X" с вашего сервиса,
который запушил метрики в pushgateway, а при значении true будут отображаться только лэйблы вашего сервиса (опять же, если будет конфликт).





Пример мониторинга python скрипта:

https://www.robustperception.io/monitoring-batch-jobs-in-python

Устанавливаем бибилиотеку для python
pip install prometheus_client

Далее выполняем скрипт
from prometheus_client import Gauge,CollectorRegistry,pushadd_to_gateway
import os, re


registry = CollectorRegistry()
duration = Gauge('mybatchjob_duration_seconds','Duration of batch job', registry=registry)
try:
 with duration.time():
 pass
 line = "localhost"
 r = "".join(os.popen("ping -c 1 " + line).readlines())
print (r)

except: pass
 else:
   last_success = Gauge('mybatchjob_last_success','Unixtime my batch job last succeeded', registry=registry)
   last_success.set_to_current_time()
finally:
pushadd_to_gateway('localhost:9091', job='my_batch_job', registry=registry)

