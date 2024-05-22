Последние версии доступны в [GITHUB](https://github.com/prometheus-community/postgres_exporter/releases)

Проверяем версию архитектуры
````
>arch
x86_64
````

Выдаем права на директорию
>sudo setfacl -m user:postgres:rwx /opt

Переходим в каталог с временными файлами
>cd /opt

Скачиваем файл
>curl -O -L  https://github.com/prometheus-community/postgres_exporter/releases/download/v0.15.0/postgres_exporter-0.15.0.linux-amd64.tar.gz

Распаковка архива
>tar -xzvf postgres_exporter-0.15.0.linux-amd64.tar.gz

Перемещаем бинарный фал в ````/usr/local/bin/````
>sudo mv /opt/postgres_exporter-0.15.0.linux-amd64/postgres_exporter /usr/local/bin/

Переходим в каталог с бинарником
>cd /usr/local/bin/


sudo chown -R postgres:postgres /usr/local/bin/postgres_exporter
sudo chmod 775 /usr/local/bin/postgres_exporter

Для тестирования конфига запуск из консоли
>DATA_SOURCE_NAME="postgresql://postgres:oracle@127.0.0.1:5432?sslmode=disable" ./postgres_exporter --auto-discover-databases --disable-default-metrics --disable-settings-metrics 

Произвольные запросы из файла ````/etc/postgres_exporter/query.yaml````
>DATA_SOURCE_NAME="postgresql://postgres:oracle@127.0.0.1:5432?sslmode=disable" ./postgres_exporter --extend.query-path=/etc/postgres_exporter/query.yaml --auto-discover-databases

Создаем службу
>sudo vi /etc/systemd/system/postgres_exporter.service

````
[Unit]
Description=PostgreSQL Exporter
After=network.target

[Service]
Type=simple
Restart=always
User=postgres
Group=postgres
Environment='DATA_SOURCE_NAME=postgresql://postgres:oracle@127.0.0.1:5432?sslmode=disable'
ExecStart=/usr/local/bin/postgres_exporter --auto-discover-databases --disable-default-metrics --disable-settings-metrics

[Install]
WantedBy=multi-user.target
````


Настройка службы
>sudo systemctl daemon-reload
>sudo systemctl enable postgres_exporter.service
>sudo systemctl start postgres_exporter.service
>sudo systemctl status postgres_exporter.service

Открываем порт 9187
>sudo firewall-cmd --zone=public --add-port=9187/tcp --permanent
>sudo systemctl restart firewalld


Если нет возможности использовать пользователя POSTGRES 
````
CREATE USER postgres_exporter PASSWORD 'password';
ALTER USER postgres_exporter SET SEARCH_PATH TO postgres_exporter,pg_catalog;

-- If deploying as non-superuser (for example in AWS RDS)
-- GRANT postgres_exporter TO :MASTER_USER;
CREATE SCHEMA postgres_exporter AUTHORIZATION postgres_exporter;

CREATE VIEW postgres_exporter.pg_stat_activity
AS
SELECT * from pg_catalog.pg_stat_activity;

GRANT SELECT ON postgres_exporter.pg_stat_activity TO postgres_exporter;

CREATE VIEW postgres_exporter.pg_stat_replication AS
SELECT * from pg_catalog.pg_stat_replication;

GRANT SELECT ON postgres_exporter.pg_stat_replication TO postgres_exporter;
````