# ELK
Установка Elasticsearch
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
```
```
sudo apt-get install apt-transport-https
```
```
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
```
```
sudo apt-get update && sudo apt-get install elasticsearch
```
После установки будет показан пароль встроенной учётной записи elastic с полными правами.

После установки добавляем elasticsearch в автозагрузку и запускаем.
```
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
sudo systemctl start elasticsearch.service
sudo systemctl status elasticsearch.service
```
Проверка работы
```
curl -k --user elastic:'ktNBdiTxO3hKqASVvHyu' https://127.0.0.1:9200
```
Установка Kibana
```
sudo apt install kibana
```
Добавляем Кибана в автозагрузку и запускаем:
```
sudo systemctl daemon-reload
sudo systemctl enable kibana.service
sudo systemctl start kibana.service
sudo systemctl status kibana.service
```
```
sudo cp -R /etc/elasticsearch/certs /etc/kibana
sudo chown -R root:kibana /etc/kibana/certs
```
Далее нам нужно настроить аутентификацию в elasticsearch. Для этого создадим пароль для встроенного пользователя kibana_system:
```
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u kibana_system
```
Сохраняем пароль и добавляем учётку в конфигурацию kibana:
```
sudo nano /etc/kibana/kibana.yml
```
```
elasticsearch.hosts: ["https://localhost:9200"]
elasticsearch.username: "kibana_system"
elasticsearch.password: "mtbg5rIwh*E*eWuBCECG"
elasticsearch.ssl.certificateAuthorities: [ "/etc/kibana/certs/http_ca.crt" ]
#server.host: "0.0.0.0" чтобы она слушала все интерфейсы либо проксировать через веб сервер на локалхост
```
```
sudo systemctl restart kibana.service
```
Установка и настройка Logstash
```
sudo apt install logstash
```
```
sudo systemctl enable logstash.service
```
```
sudo cp -R /etc/elasticsearch/certs /etc/logstash
```
```
sudo chown -R root:logstash /etc/logstash/certs
```
```
sudo nano /etc/logstash/conf.d/example.yml
```
```
input {
  beats {
    port => 5044
  }
}

filter {

}

output {
        elasticsearch {
            hosts    => "https://localhost:9200"
            index    => "websrv-%{+YYYY.MM}"
            user => "elastic"
            password => "pwd"
            cacert => "/etc/logstash/certs/http_ca.crt"
        }
}
```
```
sudo systemctl start logstash.service
```
https://serveradmin.ru/ustanovka-i-nastroyka-elasticsearch-logstash-kibana-elk-stack/
