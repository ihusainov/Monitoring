#Prometheus bot

Get prometheus bot from https://github.com/inCaller/prometheus_bot
Create folder "/etc/prometheus_bot"
Change permission to user and group "prometheus" to folder "/etc/prometheus_bot"
  $sudo chmod -R prometheus: /etc/prometheus_bot
Edit files "/etc/prometheus_bot/config.yaml" and "/etc/prometheus_bot/template.yaml"
Start prometheus bot "sudo systemctl daemon-reload" and "sudo systemctl start prometheus_bot"
Check bot is started "sudo netstat -tnlp | grep prometheus_bo"
