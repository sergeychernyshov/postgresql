Последние версии доступны в [GITHUB](https://github.com/prometheus/node_exporter/releases)

Проверяем версию архитектуры
````
>arch
x86_64
````

x86-64/amd64

Выдаем права на директорию
>sudo setfacl -m user:postgres:rwx /opt

Переходим в каталог с временными файлами
>cd /opt 

Скачиваем файл
>curl -O -L  https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz

Распаковка архива
>tar -xzvf node_exporter-1.8.1.linux-amd64.tar.gz

Перемещаем бинарный фал в ````/usr/local/bin/````
>sudo mv node_exporter-1.8.1.linux-amd64/node_exporter /usr/local/bin/

Выдаем права для исполнения
>sudo chmod 775 /usr/local/bin/node_exporter

Создаем пользователя
>sudo useradd -rs /bin/false nodeusr

Создаем файл для службы
>sudo vi /etc/systemd/system/node_exporter.service

````
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=nodeusr
Group=nodeusr
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
````

Настраиваем запуск службы
>cd /etc/systemd/system/
>sudo systemctl daemon-reload
>sudo systemctl enable node_exporter.service
>sudo systemctl start node_exporter.service
>sudo systemctl status node_exporter.service

Открываем порт 9100
>sudo firewall-cmd --zone=public --add-port=9100/tcp --permanent
>sudo systemctl restart firewalld