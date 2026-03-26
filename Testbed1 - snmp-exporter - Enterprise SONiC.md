<p align="center">
  <i>Testbed1: SNMP Exporter Test on Enterprise SONiC (Broadcom) Platform</i>
</p>

<p align="center">
  Built by <b>Jason S</b>
</p>

<p align="center">
  <a href="https://youtube.com/你的链接">
    <img src="https://img.shields.io/badge/▶ Watch Demo-YouTube-red?style=for-the-badge&logo=youtube"/>
  </a>
  <a href="https://github.com/你的repo">
    <img src="https://img.shields.io/badge/GitHub-Repo-black?style=for-the-badge&logo=github"/>
  </a>
</p>

---

<p align="center">
  <b>Collect SONiC switch metrics via SNMP Exporter and visualize in Grafana</b>
</p>

## 📌 Background

In many real-world scenarios, enterprise customers operate relatively small SONiC-based networks (typically 10–20 switches).

For these environments:

- Deploying a full telemetry stack (Kafka, streaming pipelines, etc.) may be too heavy
- A lightweight, easy-to-deploy monitoring solution is preferred

This test explores a practical approach:

👉 Running **snmp-exporter directly on the SONiC switch (via Docker)**  
👉 Exposing metrics on port **9116**  
👉 Allowing **Prometheus to scrape switches directly as monitoring targets**


## 🎯 Objective

- Validate whether SNMP-based monitoring is sufficient for small-scale SONiC deployments
- Evaluate performance impact when running snmp-exporter on-box
- Measure observability coverage (interfaces, counters, errors)

## 🧪 What This Test Covers

- SNMP agent behavior on Enterprise SONiC
- snmp-exporter deployment (Docker on switch)
- Prometheus scrape configuration
- Interface-level metrics (if_mib)
- Performance and resource impact

## 🏗️ Architecture

```
SONiC Switch
  └── snmpd (UDP 161)
        └──► snmp-exporter (Docker on-box, port 9116)
                └──► Prometheus (scrape via HTTP)
                        └──► Grafana Dashboard
```

---

## DEMO & Results

## 1️⃣ Micas W6510-32C Switch m65_1$ SONiC Switch — Enable SNMP

```bash
sonic-cli
sonic# configure t
sonic(config)# snmp-server community public
sonic(config)# exit
```

---

## 2️⃣ SONiC Switch — Deploy snmp-exporter (Docker on Micas W6510-32C Switch m65_1$)

### 2.1 Create config directory and snmp.yml - Run SNMP-Exporter on Switch

```bash
m65_1$ mkdir /opt/snmp-exporter
m65_1$ cd /opt/snmp-exporter
m65_1$ vi snmp.yml
```

Paste the following config:

### snmp.yml

<!-- SNMP_YML_START -->
<!-- SNMP_YML_END -->

### 2.2 Run Micas W6510-32C Switch m65_1$ snmp-exporter container

```bash
docker run -d \
  --name snmp-exporter \
  --restart always \
  -p 9116:9116 \
  -v /opt/snmp-exporter/snmp.yml:/etc/snmp_exporter/snmp.yml \
  prom/snmp-exporter
```

### 2.3 Verify

```bash
curl "http://<SWITCH_IP>:9116/snmp?target=<SWITCH_IP>&module=sonic_if&auth=public_v2"
```

Expected output (first few lines):

```
# HELP ifAdminStatus The desired state of the interface - 1.3.6.1.2.1.2.2.1.7
# TYPE ifAdminStatus gauge
ifAdminStatus{ifAlias="",ifDescr="Ethernet0",ifIndex="1",ifName="Ethernet0"} 2
ifAdminStatus{ifAlias="",ifDescr="Ethernet100",ifIndex="101",ifName="Ethernet100"} 2
ifAdminStatus{ifAlias="",ifDescr="Ethernet104",ifIndex="105",ifName="Ethernet104"} 2
ifAdminStatus{ifAlias="",ifDescr="Ethernet108",ifIndex="109",ifName="Ethernet108"} 2
ifAdminStatus{ifAlias="",ifDescr="Ethernet112",ifIndex="113",ifName="Ethernet112"} 2
ifAdminStatus{ifAlias="",ifDescr="Ethernet116",ifIndex="117",ifName="Ethernet116"} 2
ifAdminStatus{ifAlias="",ifDescr="Ethernet12",ifIndex="13",ifName="Ethernet12"} 2
ifAdminStatus{ifAlias="",ifDescr="Ethernet120",ifIndex="121",ifName="Ethernet120"} 2
ifAdminStatus{ifAlias="",ifDescr="Ethernet124",ifIndex="125",ifName="Ethernet124"} 2
ifAdminStatus{ifAlias="",ifDescr="Ethernet16",ifIndex="17",ifName="Ethernet16"} 2
ifAdminStatus{ifAlias="",ifDescr="Ethernet20",ifIndex="21",ifName="Ethernet20"} 2
ifAdminStatus{ifAlias="",ifDescr="Ethernet24",ifIndex="25",ifName="Ethernet24"} 2
ifAdminStatus{ifAlias="",ifDescr="Ethernet28",ifIndex="29",ifName="Ethernet28"} 2
ifAdminStatus{ifAlias="",ifDescr="Ethernet32",ifIndex="3
...
```

You can see the docker status snmp-exporter system performance and resources
<img width="898" height="45" alt="image" src="https://github.com/user-attachments/assets/0ffe4ebe-b3e3-484b-9fa9-cc271ff9572f" />

when you run snmp walk  snmpwalk -v2c -c public 192.168.xxx.x
<img width="1329" height="903" alt="image" src="https://github.com/user-attachments/assets/6fe82728-9b85-4226-91f5-62364e1ff3b0" />
<img width="1329" height="51" alt="image" src="https://github.com/user-attachments/assets/e2542dc8-fd63-472f-9163-c4eb0fd0b0b0" />
</br>
Performance CPU 0.00%, MEM 0.6% so it was not very occupied and just little resources needed of MEM.

---

## 3️⃣ Prometheus Server — Configuration

### 3.1 prometheus.yml - Prometheus Server

Location: `/etc/prometheus/prometheus.yml` (or your mounted path)

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - /etc/prometheus/rules.yml

scrape_configs:
  - job_name: sonic_interfaces
    static_configs:
      - targets:
          - <SWITCH_IP>
    metrics_path: /snmp
    params:
      module: [sonic_if]
      auth: [public_v2]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: <SWITCH_IP>:9116

  - job_name: sonic_system
    static_configs:
      - targets: [<SWITCH_IP>]
    metrics_path: /snmp
    params:
      module: [sonic_system]
      auth: [public_v2]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: <SWITCH_IP>:9116

  - job_name: sonic_sensors
    static_configs:
      - targets: [<SWITCH_IP>]
    metrics_path: /snmp
    params:
      module: [sonic_sensors]
      auth: [public_v2]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: <SWITCH_IP>:9116

  - job_name: sonic_hardware
    static_configs:
      - targets: [<SWITCH_IP>]
    metrics_path: /snmp
    params:
      module: [sonic_hardware]
      auth: [public_v2]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: <SWITCH_IP>:9116

  - job_name: sonic_ddm
    static_configs:
      - targets: [<SWITCH_IP>]
    metrics_path: /snmp
    params:
      module: [sonic_ddm]
      auth: [public_v2]
    scrape_interval: 60s
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: <SWITCH_IP>:9116

  - job_name: sonic_tables
    static_configs:
      - targets: [<SWITCH_IP>]
    metrics_path: /snmp
    params:
      module: [sonic_tables]
      auth: [public_v2]
    scrape_interval: 60s
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: <SWITCH_IP>:9116

  - job_name: sonic_routing
    static_configs:
      - targets: [<SWITCH_IP>]
    metrics_path: /snmp
    params:
      module: [sonic_routing]
      auth: [public_v2]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: <SWITCH_IP>:9116
```

> ⚠️ Replace `<SWITCH_IP>` with your actual switch IP (e.g. `100.105.8.4`)

### 3.2 rules.yml - Prometheus Server

Location: `/etc/prometheus/rules.yml`

```yaml
groups:
  - name: sonic_table_counts
    interval: 60s
    rules:
      - record: sonic_arp_count
        expr: count(ipNetToPhysicalType{instance="<SWITCH_IP>"})
      # sonic_mac_count: uncomment when MAC table data is available
      # sonic_vlan_count: uncomment when VLAN data is available
      # sonic_ipv4_route_count: uncomment when route table data is available

  - name: sonic_alerts
    rules:
      - alert: InterfaceDown
        expr: ifOperStatus{instance="<SWITCH_IP>", ifName!="eth0"} == 2
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "Interface {{ $labels.ifName }} is down"

      - alert: CPUHigh
        expr: (100 - ucdCpuIdle{instance="<SWITCH_IP>"}) > 90
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "CPU usage above 90%: {{ $value }}%"

      - alert: MemoryHigh
        expr: (1 - ucdMemAvailReal{instance="<SWITCH_IP>"} / ucdMemTotalReal{instance="<SWITCH_IP>"}) * 100 > 85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Memory usage above 85%: {{ $value }}%"

      - alert: PSUDown
        expr: psu1PowerStatus{instance="<SWITCH_IP>"} != 1 or psu2PowerStatus{instance="<SWITCH_IP>"} != 1
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "PSU power status abnormal"

      - alert: BGPPeerDown
        expr: bgpPeerState{instance="<SWITCH_IP>"} != 6
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "BGP peer {{ $labels.bgpPeerRemoteAddr }} not in established state"

      - alert: TempHigh
        expr: lmTempSensorsValue{instance="<SWITCH_IP>"} / 1000 > 75
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Temperature sensor {{ $labels.lmTempSensorsIndex }} above 75C: {{ $value }}C"

      - alert: FanDown
        expr: fan1Status{instance="<SWITCH_IP>"} != 1 or fan2Status{instance="<SWITCH_IP>"} != 1
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Fan status abnormal"

      - alert: SnmpScrapeDown
        expr: up{job=~"sonic_.*"} == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "SNMP scrape failed for job {{ $labels.job }} instance {{ $labels.instance }}"
```

### 3.3 Apply configuration - Prometheus Server

```bash
# If running as Docker container
docker restart prometheus

# Or reload without restart
curl -X POST http://localhost:9090/-/reload

# Verify targets are UP
# Go to: http://<PROMETHEUS_IP>:9090/targets
```

- You can see all modules up!
<img width="1881" height="766" alt="image" src="https://github.com/user-attachments/assets/f5a989bd-2790-4a88-b9ba-55508c2eacb8" />


---

## 4️⃣ Grafana — Dashboard Setup 

### 4.1 Add Prometheus data source

1. Go to **Connections → Data Sources → Add data source**
2. Select **Prometheus**
3. Set URL: `http://<PROMETHEUS_IP>:9090`
4. Click **Save & Test**

Get the datasource UID:

```bash
curl -s -u admin:admin http://<GRAFANA_IP>:3000/api/datasources | python3 -m json.tool | grep '"uid"'
```

### 4.2 Import dashboard via API

Replace `<DS_UID>` with your Prometheus datasource UID, then push:

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -u admin:admin \
  http://<GRAFANA_IP>:3000/api/dashboards/db \
  -d @dashboard.json
```
Download Dashboard [Dashboard](https://github.com/qwert22356/Enterprise-SONiC-Telemetry-Monitoring/blob/main/Prometheus%20Resources/dashboard.json)

<img width="1575" height="868" alt="image" src="https://github.com/user-attachments/assets/0bc4cd4b-07b8-497e-8219-c6be9089a53d" />

---

The dashboard includes the following panels:

| Panel | Metrics |
|-------|---------|
| Interface Status | `ifOperStatus` — UP/DOWN color-coded per port |
| CPU Usage % | `ucdCpuIdle/User/System/IoWait` |
| Memory Usage % | `ucdMemAvailReal / ucdMemTotalReal` |
| Temperature Sensors | `lmTempSensorsValue / 1000` |
| PSU Status | `psu1/2PowerStatus` — color-mapped |
| Fan Status | `fan1Status / psu1/2FanStatus` |
| Fan Speed (RPM) | `fan1Rpm / lmFanSensorsValue` |
| PSU Temperature | `psu1/2Temp / 1000` |
| OSPF Status | `ospfAdminStat` |
| Interface RX/TX Traffic | `ifHCInOctets / ifHCOutOctets` (bps) |
| Interface RX/TX Packets | Ucast + Mcast + Bcast breakdown (pps) |

---

## 📊 Module Coverage Summary

| Module | MIB | Key Metrics |
|--------|-----|-------------|
| `sonic_if` | IF-MIB / ifXTable | Interface status, traffic counters, errors |
| `sonic_system` | UCD-SNMP / HR-MIB | CPU, memory, uptime |
| `sonic_sensors` | LM-SENSORS / Entity Sensor | Temperature, fan RPM, voltage |
| `sonic_hardware` | Entity Sensor / Cisco MIB | PSU status, PSU fan, PSU temp/voltage |
| `sonic_ddm` | Entity Sensor | SFP TxPower, RxPower, Bias, Temp |
| `sonic_tables` | IP-MIB | ARP/Neighbor table entries |
| `sonic_routing` | BGP4-MIB / OSPF-MIB | BGP peer state, OSPF admin status |

---

## 🔍 Troubleshooting

```bash
# Test snmp-exporter manually
curl "http://<SWITCH_IP>:9116/snmp?target=<SWITCH_IP>&module=sonic_if&auth=public_v2"

# Check snmp-exporter logs
docker logs snmp-exporter --tail 20

# Verify SNMP is responding on the switch
snmpwalk -v2c -c public <SWITCH_IP> 1.3.6.1.2.1.1.1.0

# Check Prometheus rules loaded correctly
curl -s http://localhost:9090/api/v1/rules | python3 -m json.tool | grep '"name"'

# Check all targets status
curl -s http://localhost:9090/api/v1/targets | python3 -m json.tool | grep -E '"health"|"job"'
```

---

## 📝 Notes

- snmp-exporter v0.26+ requires the `auths:` block; older versions do not support it
- SONiC does not implement `dot1qTpFdbTable` (MAC) or `dot1qVlanStaticTable` (VLAN) on all platforms — verify with `snmpwalk` before adding to config
- DDM sensor OIDs use Entity Sensor MIB instance IDs that are platform-specific; walk `1.3.6.1.2.1.99.1.1.1.4` to discover all available sensors
- Temperature values from `lmTempSensorsValue` are in milli-Celsius — divide by 1000 in Grafana or recording rules
- BGP and OSPF data only appears when the protocols are configured and active on the switch
- `InterfaceDown` alert is suppressed for `eth0` (management interface) by design



