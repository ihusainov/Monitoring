Prometheus bot

Get prometheus bot from https://github.com/inCaller/prometheus_bot
Create folder "/etc/prometheus_bot"
Change permission to user and group "prometheus" to folder "/etc/prometheus_bot"
  $sudo chown -R prometheus: /etc/prometheus_bot
Edit files "/etc/prometheus_bot/config.yaml" and "/etc/prometheus_bot/template.yaml"
Start prometheus bot "sudo systemctl daemon-reload" and "sudo systemctl start prometheus_bot"
Check bot is started "sudo netstat -tnlp | grep prometheus_bo"



Get the channel id
https://gist.github.com/mraaroncruz/e76d19f7d61d59419002db54030ebe35

To get the channel id
Create your bot with botfather
Make you bot an admin of your channel
New improved next steps
Go to https://web.telegram.org
Click on your channel
Look at the URL and find the part that looks like c12112121212_17878787878787878
Remove the underscore and after c12112121212
Remove the prefixed letter 12112121212
Prefix with a -100 so -10012112121212
That's your channel id.
Old yucky next steps
Make your channel public
Create a public link
Send a message from console to @[your_public_link_text]
Copy chat id from response in console as the channel id
