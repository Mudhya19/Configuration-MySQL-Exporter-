# MySQL Exporter Installation and Configuration Guide

This guide describes how to install and configure `mysqld_exporter` on a Linux server for monitoring with Prometheus and Grafana.

## 1. Install MySQL Exporter

Navigate to the temporary directory:

```bash
cd /tmp
```

Download the exporter:

```bash
wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.17.2/mysqld_exporter-0.17.2.linux-amd64.tar.gz
```

Extract and move the binary to the desired directory:

```bash
tar vxf mysqld_exporter-0.17.2.linux-amd64.tar.gz
mv mysqld_exporter-0.17.2.linux-amd64/mysqld_exporter /home/slemp/server/mysql/bin/
chmod +x /home/slemp/server/mysql/bin/mysqld_exporter
```

## 2. Create MySQL User for Exporter

Access MySQL as root:

```bash
/home/slemp/server/mysql/bin/mysql -u root -p
```

Then execute the following SQL commands:

```sql
CREATE USER 'exporter'@'::1' IDENTIFIED BY 'password_exporter';
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'::1';
FLUSH PRIVILEGES;
```

Replace `password_exporter` with a secure password.

## 3. Create `.my.cnf` Configuration File

Create the credentials file:

```bash
nano /home/slemp/server/mysql/.my.cnf
```

With the following content:

```
[client]
user=exporter
password=password_exporter
```

Secure the file:

```bash
chmod 600 /home/slemp/server/mysql/.my.cnf
```

## 4. Run Exporter Manually (for Testing)

```bash
/home/slemp/server/mysql/bin/mysqld_exporter --config.my-cnf=/home/slemp/server/mysql/.my.cnf
```

## 5. Create a Systemd Service

Create the service file:

```bash
nano /etc/systemd/system/mysqld_exporter.service
```

Add the following content:

```ini
[Unit]
Description=MySQLd Exporter
After=network.target

[Service]
User=root
ExecStart=/home/slemp/server/mysql/bin/mysqld_exporter --config.my-cnf=/home/slemp/server/mysql/.my.cnf --web.listen-address="0.0.0.0:9104"
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable and start the service:

```bash
systemctl daemon-reload
systemctl enable --now mysqld_exporter
systemctl start mysqld_exporter
systemctl status mysqld_exporter
```

## 6. Verify Metrics

Check if the exporter is exposing metrics:

```bash
curl http://192.168.11.72:9104/metrics
```

* Prometheus: [http://192.168.11.72:9090](http://192.168.11.72:9090)
* Grafana: [http://192.168.11.72:3000](http://192.168.11.72:3000)

## 7. Grafana Dashboard Import

You can import MySQL monitoring dashboards using the following IDs:

* `7362`
* `14057`

These dashboards will visualize the metrics exported by `mysqld_exporter` and collected by Prometheus.
