# Avail Monitoring

The purpose of this repo is to assist validators or inidividuals who are running Avail Nodes that is part of the Polygon ecosystem 


## Install Prometheus & Node Exporter Software

Create the users needed
```
sudo useradd --no-create-home --shell /usr/sbin/nologin prometheus
sudo useradd --no-create-home --shell /usr/sbin/nologin node_exporter
```

Get the software needed and Extract
```
cd ~
wget https://github.com/prometheus/prometheus/releases/download/v2.26.0/prometheus-2.26.0.linux-amd64.tar.gz
tar xfz prometheus-*.tar.gz
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz 
tar xvf node_exporter-*.tar.gz
```

Place the software in the correct place
```
cd ~
cd prometheus-*.linux-amd64
sudo cp ./prometheus /usr/local/bin/
sudo cp ./promtool /usr/local/bin/
cd ~
sudo cp ./node_exporter-*.linux-amd64/node_exporter /usr/local/bin/ 
```

Create additional directories needed
```
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```

Set the permissions
```
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

Copy the consoles and console_libraries directories to /etc/prometheus, also update permissions
```
cd ~
cd prometheus-*.linux-amd64
sudo cp -r ./consoles /etc/prometheus
sudo cp -r ./console_libraries /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
```

Now clean up
```
cd .. && rm -rf prometheus*
rm -rf node_exporter*
```

Create a systemd file to start the prometheus service
```
sudo nano /etc/systemd/system/prometheus.service
```

Add the following content
```
[Unit]
  Description=Prometheus Monitoring
  Wants=network-online.target
  After=network-online.target

[Service]
  User=prometheus
  Group=prometheus
  Type=simple
  ExecStart=/usr/local/bin/prometheus \
  --config.file /etc/prometheus/prometheus.yml \
  --storage.tsdb.path /var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries
  ExecReload=/bin/kill -HUP $MAINPID

[Install]
  WantedBy=multi-user.target
```

Repeat for node exporter
```
sudo nano /etc/systemd/system/node_exporter.service
```

Add the following content 
```
[Unit]
  Description=Node Exporter
  Wants=network-online.target
  After=network-online.target

[Service] 
  User=node_exporter
  Group=node_exporter
  Type=simple
  ExecStart=/usr/local/bin/node_exporter

[Install]
  WantedBy=multi-user.target
```

Start the services and enable auto restart
```
sudo systemctl daemon-reload && systemctl enable prometheus && systemctl start prometheus
sudo systemctl daemon-reload && systemctl enable node_exporter && systemctl start node_exporter
```

Ensure the services are running fine

## Configure prometheus & Node Exporter

Edit the configuration file
```
sudo nano /etc/prometheus/prometheus.yml
```

Add the below in the file
```
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  # - "first.rules"
  # - "second.rules"

scrape_configs:
  - job_name: "prometheus"
    scrape_interval: 5s
    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "substrate_node"
    scrape_interval: 5s
    static_configs:
      - targets: ["localhost:9615"]
  - job_name: node
    static_configs:
      - targets: ['localhost:9100']
```

Restart Prometheus
```
systemctl restart prometheus
```

## Install Grafana

Download and install
```
sudo apt-get install -y adduser libfontconfig1
wget https://dl.grafana.com/oss/release/grafana_8.3.3_amd64.deb
sudo dpkg -i grafana_8.3.3_amd64.deb
```
If all is fine then start Grafana
```
sudo systemctl daemon-reload
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

## Import Grafana Dashbaord

You can now access it by going to the http://SERVER_IP_ADDRESS:3000/login. The default user and password is admin/admin.

You will need to setup the datasource as shown below
![image](https://user-images.githubusercontent.com/61656547/211848471-bd36c6d7-6f64-4a09-89f0-e90c767e63ab.png)

Using the import you can import the Avail Dashbaord in this repo

![image](https://user-images.githubusercontent.com/61656547/211849069-7941363c-e0e9-48ad-9a5b-852f5c5f33cc.png)

Import the dashboard
![image](https://user-images.githubusercontent.com/61656547/211849660-97ccd7cf-c03e-47a0-b977-79ed612a8246.png)

You will then be able to display the dashboard
![image](https://user-images.githubusercontent.com/61656547/211849996-fe6d3967-b081-45d9-86a3-be500d5ae66e.png)



