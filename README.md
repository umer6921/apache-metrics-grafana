# Setup alert on the status of apache using prometheus and grafana

## 1) Prerequisites
### 1) Install Apache:
Ensure Apache is installed and running on your server.

### 2) Install Prometheus:
Prometheus should be installed and running.

### 3) Install Grafana:

Grafana should be installed and running.

## 2) Set Up Apache Exporter

### 1) Enable mod_status in Apache:
Edit your Apache configuration file (typically /etc/apache2/apache2.conf or /etc/httpd/conf/httpd.conf) and add the following lines:

```
<Location /server-status>
    SetHandler server-status
    Require all granted
</Location>
```
Restart Apache to apply the changes:
```
sudo systemctl restart apache2
```
### 2) Download and Run Apache Exporter:
Download the Apache exporter binary and run it as a service
```
wget https://github.com/Lusitaniae/apache_exporter/releases/download/v0.8.0/apache_exporter-0.8.0.linux-amd64.tar.gz

tar -xzf apache_exporter-0.8.0.linux-amd64.tar.gz

sudo mv apache_exporter-0.8.0.linux-amd64/apache_exporter /usr/local/bin/
```
### 3) Create a Systemd Service Unit File

```
sudo nano /etc/systemd/system/apache_exporter.service
```

### 4) Add the Following Configuration to the Service File
```
[Unit]
Description=Prometheus Apache Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=www-data
Group=www-data
Type=simple
ExecStart=/usr/local/bin/apache_exporter --scrape_uri=http://localhost/server-status/?auto --telemetry.address=:9117
Restart=on-failure

[Install]
WantedBy=multi-user.target

```

### Explanation of the Service File
- [Unit] Section:
  
  - Description: Describes the service.

  - Wants and After: Ensure the network is up before starting the service.
- [Service] Section:
 
   - User and Group: Run the service as the www-data user and group. Adjust as needed.
   
   -  Type: Set to simple for a simple service.

  -  ExecStart: The command to start the Apache exporter.
   
  - Restart: Defines the restart policy. on-failure will restart the service if it fails.

- [Install] Section:
     - WantedBy: Specifies the target that the service should be started under. multi-user.target is a common target for most services.

### Reload Systemd Daemon
```
sudo systemctl daemon-reload
```
### Enable the Service
```
sudo systemctl enable apache_exporter.service
```
### Start the Service
```
sudo systemctl start apache_exporter.service
```

### Check the Status of the Service
```
sudo systemctl status apache_exporter.service
```

## Access the apache exporter

Open the inbound port 9117 from security

visit http://server-ip:9117/metrics

##  Configure Prometheus to Scrape Apache Exporter

### Edit Prometheus Configuration:
Add a job to scrape the Apache exporter. Edit the prometheus.yml file (typically located in /etc/prometheus/prometheus.yml or /usr/local/etc/prometheus/prometheus.yml).
```
scrape_configs:
  - job_name: 'apache'
    static_configs:
      - targets: ['server-ip:9117'] 

```

- Add prometheus public key to that server in apache2 exporter is running and open 22 port inbound for prometheus server ip

- Open the inbound security port for port 9090 for the prometheus server public ip

- Open the inbound security for port 9117 for the prometheus server public ip

```
sudo systemctl restart prometheus
```
Setup the alert on grafana....


