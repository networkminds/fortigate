# How to Monitor FortiGate with Grafana + Prometheus

### 1. Download the FortiGate Exporter

Clone the FortiGate exporter repository from GitHub:

```bash
$ sudo mkdir -p /opt/fortigate-monitoring-stack
$ cd /opt/fortigate-monitoring-stack
$ sudo git clone https://github.com/prometheus-community/fortigate_exporter.git
```
### 2. Install "fortigate_exporter" application. 

This application is written in "Go" language. We can install it using Docker file. But unfortunately to run docker image we will need credentials. So instead of docker image we will build go application manually.

If "Golang" is not installed we can install "Go" in Ubuntu OS using:
```bash
$ sudo snap install go --classic
```

Build go file to the "/usr/local/bin/" directory:
```bash
$ cd /opt/fortigate-monitoring-stack
$ sudo go build -o /usr/local/bin/fortigate_exporter .
```

Create "fortigate-key.yaml" file inside "/usr/local/bin/" directory. Using "exclude" we will exclude services which will be not monitored using Go application. In our case, for example we are not using BGP in pur fortigate, that is why we are excluding:
```bash
$ sudo vi /usr/local/bin/fortigate-key.yaml
```
```bash
"https://172.19.1.1":
  token: xxxxxxxxxxxxxxxxxxxxxxx
  # If you have a smaller fortigate unit you might want
  # to exclude sensors as they do not have any
  probes:
    exclude:
      - System/SensorInfo
      - System/HAChecksums
      - Firewall/IpPool
      - System/HAChecksums
      - System/VDOMResources
      - System/Fortimanager/Status
      - System/SDNConnector
      - User/Fsso
      - Wifi
      - Log/Fortianalyzer/Status
      - Log/Fortianalyzer/Queue
      - System/HAStatistics
      - System/LinkMonitor
      - VirtualWAN/HealthCheck
      - BGP
      - OSPF
      - Firewall/LoadBalance
      - Wifi/Clients
      - Wifi/ManagedAP
      - Switch/ManagedSwitch
```
Create "fortigate_exporter.service" systemd service:
```bash
sudo vi /etc/systemd/system/fortigate_exporter.service
```
```bash
[Unit]
Description=FortiGate Exporter
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/usr/local/bin/fortigate_exporter -auth-file "/usr/local/bin/fortigate-key.yaml" --insecure
Restart=always
RestartSec=5
User=root
WorkingDirectory=/usr/local/bin
StandardOutput=append:/var/log/fortigate_exporter.log
StandardError=append:/var/log/fortigate_exporter.log
	
[Install]
WantedBy=multi-user.target
```
Enable and run systemd unit:
```bash
$ sudo systemctl daemon-reload
$ sudo systemctl enable fortigate_exporter
$ sudo systemctl start fortigate_exporter
```

### 3. Install "Prometheus" and "Grafana" using docker-compose.yaml file:
Create "grafana" directory and inside folder create file "grafana.yaml"
```bash
cd /opt/fortigate-monitoring-stack
mkdir grafana && $ cd grafana
touch grafana.yaml
```
Create "prometheus" directory and inside directory create file "prometheus.yaml" 
```bash
cd /opt/fortigate-monitoring-stack
mkdir prometheus && $ cd prometheus
touch prometheus.yaml
```
Create "telemetry-stack" systemd unit file:
```bash
[Unit]
Description=Prometheus & Grafana Docker Compose Stack
After=network-online.target docker.service
Requires=docker.service
	
[Service]
Type=oneshot
WorkingDirectory=/opt/fortigate-monitoring-stack
ExecStart=/usr/bin/docker-compose up -d
ExecStop=/usr/bin/docker-compose down
RemainAfterExit=yes
TimeoutStartSec=0
	
[Install]
WantedBy=multi-user.target
```
Enable and run systemd unit file:
```bash
s sudo systemctl daemon-reload
$ sudo systemctl enable telemetry-stack
$ sudo systemctl start telemetry-stack
```

### 4. Create necessary permissions inside Fortigate device:
```bash
config system accprofile
    edit "monitor"
        # global scope will fail on non multi-VDOM firewall
        set scope global
        set authgrp read
        # As of FortiOS 6.2.1 it seems `fwgrp-permissions.other` is removed,
        # use 'fwgrp read' to get load balance servers metrics
        set fwgrp custom
        set loggrp custom
        set netgrp custom
        set sysgrp custom
        set vpngrp read
        set wifi read
        # will fail for most recent FortiOS
        set system-diagnostics disable
        config fwgrp-permission
            set policy read
            set others read
        end
        config netgrp-permission
            set cfg read
            set route-cfg read
        end
        config loggrp-permission
            set config read
        end
        config sysgrp-permission
            set cfg read
        end
    next
end
```

### 5. Create "REST API admin" inside Fortigate device:
<img width="551" height="402" alt="image" src="https://github.com/user-attachments/assets/450e9de5-9097-4cf8-9ca2-3026c2fc793b" />
























