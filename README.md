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

Create "fortigate-key.yaml" file inside "/usr/local/bin/" directory:
```bash
$ sudo vi /usr/local/bin/fortigate-key.yaml
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
Create systemd service:
```bash
sudo vi /etc/systemd/system/fortigate_exporter.service
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
```bash
cd /opt/fortigate-monitoring-stack
mkdir grafana && $ cd grafana


















