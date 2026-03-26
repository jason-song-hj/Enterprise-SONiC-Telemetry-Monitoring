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



<!-- SNMP_YML_START -->
```yaml
auths:
  public_v2:
    version: 2
    community: public

modules:

  # ============================================================
  # Module 1: Interface Traffic & Status
  # ============================================================
  sonic_if:
    walk:
      - 1.3.6.1.2.1.2.2      # ifTable (ifOperStatus 等)
      - 1.3.6.1.2.1.31.1.1   # ifXTable (ifName, ifHCInOctets 等)
    metrics:
      - name: ifName
        oid: 1.3.6.1.2.1.31.1.1.1.1
        type: DisplayString
        help: Interface name
        indexes:
          - labelname: ifIndex
            type: gauge

      - name: ifOperStatus
        oid: 1.3.6.1.2.1.2.2.1.8
        type: gauge
        help: "Interface operational status: 1=up, 2=down"
        indexes:
          - labelname: ifIndex
            type: gauge
        lookups:
          - labels: [ifIndex]
            labelname: ifName
            oid: 1.3.6.1.2.1.31.1.1.1.1
            type: DisplayString

      - name: ifAdminStatus
        oid: 1.3.6.1.2.1.2.2.1.7
        type: gauge
        help: "Interface admin status: 1=up, 2=down"
        indexes:
          - labelname: ifIndex
            type: gauge
        lookups:
          - labels: [ifIndex]
            labelname: ifName
            oid: 1.3.6.1.2.1.31.1.1.1.1
            type: DisplayString

      - name: ifHCInOctets
        oid: 1.3.6.1.2.1.31.1.1.1.6
        type: counter
        help: Total bytes received (64-bit)
        indexes:
          - labelname: ifIndex
            type: gauge
        lookups:
          - labels: [ifIndex]
            labelname: ifName
            oid: 1.3.6.1.2.1.31.1.1.1.1
            type: DisplayString

      - name: ifHCOutOctets
        oid: 1.3.6.1.2.1.31.1.1.1.10
        type: counter
        help: Total bytes sent (64-bit)
        indexes:
          - labelname: ifIndex
            type: gauge
        lookups:
          - labels: [ifIndex]
            labelname: ifName
            oid: 1.3.6.1.2.1.31.1.1.1.1
            type: DisplayString

      - name: ifHCInUcastPkts
        oid: 1.3.6.1.2.1.31.1.1.1.7
        type: counter
        help: Unicast packets received (64-bit)
        indexes:
          - labelname: ifIndex
            type: gauge
        lookups:
          - labels: [ifIndex]
            labelname: ifName
            oid: 1.3.6.1.2.1.31.1.1.1.1
            type: DisplayString

      - name: ifHCOutUcastPkts
        oid: 1.3.6.1.2.1.31.1.1.1.11
        type: counter
        help: Unicast packets sent (64-bit)
        indexes:
          - labelname: ifIndex
            type: gauge
        lookups:
          - labels: [ifIndex]
            labelname: ifName
            oid: 1.3.6.1.2.1.31.1.1.1.1
            type: DisplayString

      - name: ifInErrors
        oid: 1.3.6.1.2.1.2.2.1.14
        type: counter
        help: Inbound errors
        indexes:
          - labelname: ifIndex
            type: gauge
        lookups:
          - labels: [ifIndex]
            labelname: ifName
            oid: 1.3.6.1.2.1.31.1.1.1.1
            type: DisplayString

      - name: ifOutErrors
        oid: 1.3.6.1.2.1.2.2.1.20
        type: counter
        help: Outbound errors
        indexes:
          - labelname: ifIndex
            type: gauge
        lookups:
          - labels: [ifIndex]
            labelname: ifName
            oid: 1.3.6.1.2.1.31.1.1.1.1
            type: DisplayString

      - name: ifHighSpeed
        oid: 1.3.6.1.2.1.31.1.1.1.15
        type: gauge
        help: Interface speed in Mbps
        indexes:
          - labelname: ifIndex
            type: gauge
        lookups:
          - labels: [ifIndex]
            labelname: ifName
            oid: 1.3.6.1.2.1.31.1.1.1.1
            type: DisplayString

  # ============================================================
  # Module 2: CPU & Memory (UCD-SNMP MIB)
  # ============================================================
  sonic_system:
    walk:
      - 1.3.6.1.4.1.2021.11   # CPU (UCD-SNMP)
      - 1.3.6.1.4.1.2021.4    # Memory (UCD-SNMP)
      - 1.3.6.1.2.1.25.2.3    # hrStorageTable
      - 1.3.6.1.2.1.1         # sysDescr, sysUpTime
    metrics:
      # --- CPU ---
      - name: ucdCpuIdle
        oid: 1.3.6.1.4.1.2021.11.11.0
        type: gauge
        help: CPU idle percentage

      - name: ucdCpuUser
        oid: 1.3.6.1.4.1.2021.11.9.0
        type: gauge
        help: CPU user percentage

      - name: ucdCpuSystem
        oid: 1.3.6.1.4.1.2021.11.10.0
        type: gauge
        help: CPU system percentage

      - name: ucdCpuIoWait
        oid: 1.3.6.1.4.1.2021.11.54.0
        type: gauge
        help: CPU iowait percentage

      # --- Memory (UCD) ---
      - name: ucdMemTotalReal
        oid: 1.3.6.1.4.1.2021.4.5.0
        type: gauge
        help: Total real memory (kB)

      - name: ucdMemAvailReal
        oid: 1.3.6.1.4.1.2021.4.6.0
        type: gauge
        help: Available real memory (kB)

      - name: ucdMemTotalFree
        oid: 1.3.6.1.4.1.2021.4.11.0
        type: gauge
        help: Total free memory including swap (kB)

      - name: ucdMemBuffer
        oid: 1.3.6.1.4.1.2021.4.14.0
        type: gauge
        help: Memory used for buffers (kB)

      - name: ucdMemCached
        oid: 1.3.6.1.4.1.2021.4.15.0
        type: gauge
        help: Memory used for cache (kB)

      # --- hrStorage (内存占用 index=54) ---
      - name: hrStorageSize
        oid: 1.3.6.1.2.1.25.2.3.1.5
        type: gauge
        help: Storage size in allocation units
        indexes:
          - labelname: hrStorageIndex
            type: gauge

      - name: hrStorageUsed
        oid: 1.3.6.1.2.1.25.2.3.1.6
        type: gauge
        help: Storage used in allocation units
        indexes:
          - labelname: hrStorageIndex
            type: gauge

      # --- System ---
      - name: sysUpTimeInstance
        oid: 1.3.6.1.2.1.1.3.0
        type: gauge
        help: System uptime in hundredths of a second

  # ============================================================
  # Module 3: Temperature Sensors (UCD lmSensors + Entity Sensor MIB)
  # ============================================================
  sonic_sensors:
    walk:
      - 1.3.6.1.4.1.2021.13.16.2  # lmTempSensorsTable
      - 1.3.6.1.4.1.2021.13.16.3  # lmFanSensorsTable
      - 1.3.6.1.4.1.2021.13.16.4  # lmVoltSensorsTable
      - 1.3.6.1.2.1.99.1.1.1      # entPhySensorValue (温度/PSU/DDM)
    metrics:
      # UCD lmSensors 温度
      - name: lmTempSensorsValue
        oid: 1.3.6.1.4.1.2021.13.16.2.1.3
        type: gauge
        help: Temperature sensor value (milli-Celsius, divide by 1000 for C)
        indexes:
          - labelname: lmTempSensorsIndex
            type: gauge

      # UCD lmSensors 风扇
      - name: lmFanSensorsValue
        oid: 1.3.6.1.4.1.2021.13.16.3.1.3
        type: gauge
        help: Fan speed sensor value (RPM)
        indexes:
          - labelname: lmFanSensorsIndex
            type: gauge

      # UCD lmSensors 电压
      - name: lmVoltSensorsValue
        oid: 1.3.6.1.4.1.2021.13.16.4.1.3
        type: gauge
        help: Voltage sensor value (mV)
        indexes:
          - labelname: lmVoltSensorsIndex
            type: gauge

      # Entity Sensor - 温度传感器 (index 200990110~200990710)
      - name: entPhySensorValue
        oid: 1.3.6.1.2.1.99.1.1.1.4
        type: gauge
        help: Entity physical sensor value (raw)
        indexes:
          - labelname: entPhySensorIndex
            type: gauge

      # Entity Sensor - 状态
      - name: entPhySensorOperStatus
        oid: 1.3.6.1.2.1.99.1.1.1.5
        type: gauge
        help: "Entity sensor operational status: 1=ok, 2=unavailable, 3=nonoperational"
        indexes:
          - labelname: entPhySensorIndex
            type: gauge

  # ============================================================
  # Module 4: PSU / FAN / Hardware Status (Entity Sensor MIB)
  # ============================================================
  sonic_hardware:
    walk:
      - 1.3.6.1.2.1.99.1.1.1      # entPhySensorValue / Status (所有硬件传感器)
      - 1.3.6.1.4.1.9.9.117.1.1.2 # Cisco PSU power status MIB
    metrics:
      # FAN 1 转速
      - name: fan1Rpm
        oid: 1.3.6.1.2.1.99.1.1.1.4.499019920
        type: gauge
        help: Fan 1 speed (RPM)

      # FAN 1 状态
      - name: fan1Status
        oid: 1.3.6.1.2.1.99.1.1.1.5.499019920
        type: gauge
        help: "Fan 1 status: 1=ok, 2=unavailable, 3=nonoperational"

      # FAN 2 转速
      - name: fan2Rpm
        oid: 1.3.6.1.2.1.99.1.1.1.4.602019920
        type: gauge
        help: Fan 2 speed (RPM)

      # FAN 2 状态
      - name: fan2Status
        oid: 1.3.6.1.2.1.99.1.1.1.5.602019920
        type: gauge
        help: "Fan 2 status: 1=ok, 2=unavailable, 3=nonoperational"

      # PSU 1 风扇转速
      - name: psu1FanRpm
        oid: 1.3.6.1.2.1.99.1.1.1.4.601019920
        type: gauge
        help: PSU 1 fan speed (RPM)

      # PSU 1 风扇状态
      - name: psu1FanStatus
        oid: 1.3.6.1.2.1.99.1.1.1.5.601019920
        type: gauge
        help: "PSU 1 fan status: 1=ok, 2=unavailable, 3=nonoperational"

      # PSU 1 电源状态 (Cisco MIB)
      - name: psu1PowerStatus
        oid: 1.3.6.1.4.1.9.9.117.1.1.2.1.2.1
        type: gauge
        help: "PSU 1 power status: 1=normal, 2=warning, 3=critical, 4=shutdown, 5=notPresent, 6=notFunctioning"

      # PSU 1 温度
      - name: psu1Temp
        oid: 1.3.6.1.2.1.99.1.1.1.4.601240010
        type: gauge
        help: PSU 1 temperature (raw, divide by 1000 for C)

      # PSU 1 电压
      - name: psu1Voltage
        oid: 1.3.6.1.2.1.99.1.1.1.4.601240050
        type: gauge
        help: PSU 1 voltage (raw, divide by 1000 for V)

      # PSU 2 风扇转速
      - name: psu2FanRpm
        oid: 1.3.6.1.2.1.99.1.1.1.4.602019920
        type: gauge
        help: PSU 2 fan speed (RPM)

      # PSU 2 风扇状态
      - name: psu2FanStatus
        oid: 1.3.6.1.2.1.99.1.1.1.5.602019920
        type: gauge
        help: "PSU 2 fan status: 1=ok, 2=unavailable, 3=nonoperational"

      # PSU 2 电源状态
      - name: psu2PowerStatus
        oid: 1.3.6.1.4.1.9.9.117.1.1.2.1.2.2
        type: gauge
        help: "PSU 2 power status: 1=normal, 2=warning, 3=critical, 4=shutdown, 5=notPresent, 6=notFunctioning"

      # PSU 2 温度
      - name: psu2Temp
        oid: 1.3.6.1.2.1.99.1.1.1.4.602240010
        type: gauge
        help: PSU 2 temperature (raw, divide by 1000 for C)

      # PSU 2 电压
      - name: psu2Voltage
        oid: 1.3.6.1.2.1.99.1.1.1.4.602240050
        type: gauge
        help: PSU 2 voltage (raw, divide by 1000 for V)

  # ============================================================
  # Module 5: DDM Optical Transceivers (Entity Sensor MIB, full tree walk)
  # ============================================================
  sonic_ddm:
    walk:
      - 1.3.6.1.2.1.99.1.1.1.4    # entPhySensorValue (含所有DDM实例)
      - 1.3.6.1.2.1.99.1.1.1.5    # entPhySensorOperStatus
    metrics:
      - name: ddmSensorValue
        oid: 1.3.6.1.2.1.99.1.1.1.4
        type: gauge
        help: DDM sensor raw value (TxPower/RxPower/Bias/Temp，需按index换算单位)
        indexes:
          - labelname: ddmSensorIndex
            type: gauge

      - name: ddmSensorStatus
        oid: 1.3.6.1.2.1.99.1.1.1.5
        type: gauge
        help: "DDM sensor status: 1=ok, 2=unavailable, 3=nonoperational"
        indexes:
          - labelname: ddmSensorIndex
            type: gauge

  # ============================================================
  # Module 6: Table Counters (ARP / MAC / VLAN / Routes)
  # ============================================================
  sonic_tables:
    walk:
      - 1.3.6.1.2.1.4.22.1.2
      - 1.3.6.1.2.1.4.24.2
      - 1.3.6.1.2.1.4.34
      - 1.3.6.1.2.1.17.7.1.2.2
      - 1.3.6.1.2.1.17.7.1.1.4
      - 1.3.6.1.4.1.4413.1.2.2.1.1.1.1

  sonic_routing:
    walk:
      - 1.3.6.1.2.1.15.2.0    # bgpLocalAs
      - 1.3.6.1.2.1.15.3      # bgpPeerTable
      - 1.3.6.1.2.1.14.1.1.0  # ospfRouterId
      - 1.3.6.1.2.1.14.1.2.0  # ospfAdminStat
    metrics:
      - name: bgpLocalAs
        oid: 1.3.6.1.2.1.15.2.0
        type: gauge
        help: BGP local AS number

      - name: bgpPeerState
        oid: 1.3.6.1.2.1.15.3.1.2
        type: gauge
        help: "BGP peer state: 1=idle,2=connect,3=active,4=opensent,5=openconfirm,6=established"
        indexes:
          - labelname: bgpPeerRemoteAddr
            type: InetAddressIPv4

      - name: bgpPeerAdminStatus
        oid: 1.3.6.1.2.1.15.3.1.3
        type: gauge
        help: "BGP peer admin status: 1=stop, 2=start"
        indexes:
          - labelname: bgpPeerRemoteAddr
            type: InetAddressIPv4

      - name: bgpPeerInUpdates
        oid: 1.3.6.1.2.1.15.3.1.10
        type: counter
        help: BGP updates received from peer
        indexes:
          - labelname: bgpPeerRemoteAddr
            type: InetAddressIPv4

      - name: bgpPeerOutUpdates
        oid: 1.3.6.1.2.1.15.3.1.11
        type: counter
        help: BGP updates sent to peer
        indexes:
          - labelname: bgpPeerRemoteAddr
            type: InetAddressIPv4

      - name: ospfAdminStat
        oid: 1.3.6.1.2.1.14.1.2.0
        type: gauge
        help: "OSPF admin status: 1=enabled, 2=disabled"

      - name: ospfRouterId
        oid: 1.3.6.1.2.1.14.1.1.0
        type: DisplayString
        help: OSPF router ID

```
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

<!-- PROMETHEUS_YML_START -->
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
<!-- PROMETHEUS_YML_END -->

> ⚠️ Replace `<SWITCH_IP>` with your actual switch IP (e.g. `100.105.8.4`)

### 3.2 rules.yml - Prometheus Server

Location: `/etc/prometheus/rules.yml`

<!-- RULES_YML_START -->
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
<!-- RULES_YML_END -->

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

#### dashboard.json #### 
<!-- DASHBOARD_JSON_START -->
```json
{
  "dashboard": {
    "title": "SONiC Switch Monitor",
    "uid": "sonic-switch-01",
    "tags": ["sonic", "snmp", "network"],
    "timezone": "browser",
    "refresh": "30s",
    "schemaVersion": 38,
    "templating": {
      "list": [
        {
          "name": "instance",
          "type": "query",
          "label": "Switch",
          "datasource": {"type": "prometheus", "uid": "efglbr4gf9b7ka"},
          "query": "label_values(ifOperStatus, instance)",
          "refresh": 2,
          "sort": 1,
          "current": {}
        },
        {
          "name": "ifName",
          "type": "query",
          "label": "Interface",
          "datasource": {"type": "prometheus", "uid": "efglbr4gf9b7ka"},
          "query": "label_values(ifOperStatus{instance=\"$instance\", ifName!=\"eth0\"}, ifName)",
          "multi": true,
          "includeAll": true,
          "allValue": "Ethernet.*",
          "refresh": 2,
          "sort": 3,
          "current": {}
        }
      ]
    },
    "panels": [
      {
        "id": 1,
        "title": "Interface Status",
        "type": "stat",
        "gridPos": {"x": 0, "y": 0, "w": 24, "h": 5},
        "datasource": {"type": "prometheus", "uid": "efglbr4gf9b7ka"},
        "targets": [
          {
            "expr": "sort_desc(ifOperStatus{instance=\"$instance\", ifName=~\"Ethernet.*\"})",
            "legendFormat": "{{ifName}}",
            "instant": true
          }
        ],
        "options": {
          "reduceOptions": {"calcs": ["lastNotNull"]},
          "colorMode": "background",
          "graphMode": "none",
          "textMode": "name",
          "orientation": "auto"
        },
        "fieldConfig": {
          "defaults": {
            "mappings": [
              {
                "type": "value",
                "options": {
                  "1": {"text": "UP", "color": "green", "index": 0},
                  "2": {"text": "DOWN", "color": "red", "index": 1}
                }
              }
            ],
            "thresholds": {"mode": "absolute", "steps": [{"color": "green", "value": null}]}
          }
        }
      },
      {
        "id": 2,
        "title": "CPU Usage %",
        "type": "timeseries",
        "gridPos": {"x": 0, "y": 5, "w": 8, "h": 7},
        "datasource": {"type": "prometheus", "uid": "efglbr4gf9b7ka"},
        "targets": [
          {"expr": "100 - ucdCpuIdle{instance=\"$instance\"}", "legendFormat": "Total"},
          {"expr": "ucdCpuUser{instance=\"$instance\"}", "legendFormat": "User"},
          {"expr": "ucdCpuSystem{instance=\"$instance\"}", "legendFormat": "System"},
          {"expr": "ucdCpuIoWait{instance=\"$instance\"}", "legendFormat": "IoWait"}
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percent", "min": 0, "max": 100,
            "custom": {"lineWidth": 1, "fillOpacity": 10},
            "thresholds": {"mode": "absolute", "steps": [
              {"color": "green", "value": null},
              {"color": "orange", "value": 80},
              {"color": "red", "value": 90}
            ]}
          }
        }
      },
      {
        "id": 3,
        "title": "Memory Usage %",
        "type": "timeseries",
        "gridPos": {"x": 8, "y": 5, "w": 8, "h": 7},
        "datasource": {"type": "prometheus", "uid": "efglbr4gf9b7ka"},
        "targets": [
          {
            "expr": "(1 - ucdMemAvailReal{instance=\"$instance\"} / ucdMemTotalReal{instance=\"$instance\"}) * 100",
            "legendFormat": "Used %"
          },
          {
            "expr": "ucdMemBuffer{instance=\"$instance\"} / ucdMemTotalReal{instance=\"$instance\"} * 100",
            "legendFormat": "Buffer %"
          },
          {
            "expr": "ucdMemCached{instance=\"$instance\"} / ucdMemTotalReal{instance=\"$instance\"} * 100",
            "legendFormat": "Cached %"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percent", "min": 0, "max": 100,
            "custom": {"lineWidth": 1, "fillOpacity": 10},
            "thresholds": {"mode": "absolute", "steps": [
              {"color": "green", "value": null},
              {"color": "orange", "value": 75},
              {"color": "red", "value": 85}
            ]}
          }
        }
      },
      {
        "id": 4,
        "title": "Temperature Sensors (C)",
        "type": "timeseries",
        "gridPos": {"x": 16, "y": 5, "w": 8, "h": 7},
        "datasource": {"type": "prometheus", "uid": "efglbr4gf9b7ka"},
        "targets": [
          {
            "expr": "lmTempSensorsValue{instance=\"$instance\"} / 1000",
            "legendFormat": "Sensor {{lmTempSensorsIndex}}"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "celsius",
            "custom": {"lineWidth": 1, "fillOpacity": 10},
            "thresholds": {"mode": "absolute", "steps": [
              {"color": "green", "value": null},
              {"color": "orange", "value": 75},
              {"color": "red", "value": 85}
            ]}
          }
        }
      },
      {
        "id": 5,
        "title": "PSU Status",
        "type": "stat",
        "gridPos": {"x": 0, "y": 12, "w": 6, "h": 4},
        "datasource": {"type": "prometheus", "uid": "efglbr4gf9b7ka"},
        "targets": [
          {"expr": "psu1PowerStatus{instance=\"$instance\"}", "legendFormat": "PSU 1", "instant": true},
          {"expr": "psu2PowerStatus{instance=\"$instance\"}", "legendFormat": "PSU 2", "instant": true}
        ],
        "options": {
          "reduceOptions": {"calcs": ["lastNotNull"]},
          "colorMode": "background",
          "graphMode": "none"
        },
        "fieldConfig": {
          "defaults": {
            "mappings": [
              {
                "type": "value",
                "options": {
                  "1": {"text": "Normal", "color": "green", "index": 0},
                  "2": {"text": "Warning", "color": "orange", "index": 1},
                  "3": {"text": "Critical", "color": "red", "index": 2},
                  "4": {"text": "Shutdown", "color": "red", "index": 3},
                  "5": {"text": "Not Present", "color": "grey", "index": 4},
                  "6": {"text": "Not Functioning", "color": "red", "index": 5}
                }
              }
            ]
          }
        }
      },
      {
        "id": 6,
        "title": "Fan Status",
        "type": "stat",
        "gridPos": {"x": 6, "y": 12, "w": 6, "h": 4},
        "datasource": {"type": "prometheus", "uid": "efglbr4gf9b7ka"},
        "targets": [
          {"expr": "fan1Status{instance=\"$instance\"}", "legendFormat": "Fan 1", "instant": true},
          {"expr": "psu1FanStatus{instance=\"$instance\"}", "legendFormat": "PSU1 Fan", "instant": true},
          {"expr": "psu2FanStatus{instance=\"$instance\"}", "legendFormat": "PSU2 Fan", "instant": true}
        ],
        "options": {
          "reduceOptions": {"calcs": ["lastNotNull"]},
          "colorMode": "background",
          "graphMode": "none"
        },
        "fieldConfig": {
          "defaults": {
            "mappings": [
              {
                "type": "value",
                "options": {
                  "1": {"text": "OK", "color": "green", "index": 0},
                  "2": {"text": "Unavailable", "color": "orange", "index": 1},
                  "3": {"text": "Failed", "color": "red", "index": 2}
                }
              }
            ]
          }
        }
      },
      {
        "id": 7,
        "title": "Fan Speed (RPM)",
        "type": "bargauge",
        "gridPos": {"x": 12, "y": 12, "w": 6, "h": 4},
        "datasource": {"type": "prometheus", "uid": "efglbr4gf9b7ka"},
        "targets": [
          {"expr": "fan1Rpm{instance=\"$instance\"}", "legendFormat": "Fan 1", "instant": true},
          {"expr": "psu1FanRpm{instance=\"$instance\"}", "legendFormat": "PSU1 Fan", "instant": true},
          {"expr": "psu2FanRpm{instance=\"$instance\"}", "legendFormat": "PSU2 Fan", "instant": true},
          {"expr": "lmFanSensorsValue{instance=\"$instance\"}", "legendFormat": "Sensor {{lmFanSensorsIndex}}", "instant": true}
        ],
        "options": {
          "reduceOptions": {"calcs": ["lastNotNull"]},
          "orientation": "horizontal",
          "displayMode": "gradient"
        },
        "fieldConfig": {
          "defaults": {
            "unit": "rotrpm",
            "thresholds": {"mode": "absolute", "steps": [
              {"color": "green", "value": null},
              {"color": "red", "value": 0}
            ]}
          }
        }
      },
      {
        "id": 8,
        "title": "PSU Voltage & Temp",
        "type": "stat",
        "gridPos": {"x": 18, "y": 12, "w": 6, "h": 4},
        "datasource": {"type": "prometheus", "uid": "efglbr4gf9b7ka"},
        "targets": [
          {"expr": "psu1Temp{instance=\"$instance\"} / 1000", "legendFormat": "PSU1 Temp C", "instant": true},
          {"expr": "psu2Temp{instance=\"$instance\"} / 1000", "legendFormat": "PSU2 Temp C", "instant": true}
        ],
        "options": {
          "reduceOptions": {"calcs": ["lastNotNull"]},
          "colorMode": "value",
          "graphMode": "none"
        },
        "fieldConfig": {
          "defaults": {"unit": "celsius"}
        }
      },
      {
        "id": 9,
        "title": "OSPF Status",
        "type": "stat",
        "gridPos": {"x": 0, "y": 16, "w": 6, "h": 4},
        "datasource": {"type": "prometheus", "uid": "efglbr4gf9b7ka"},
        "targets": [
          {"expr": "ospfAdminStat{instance=\"$instance\"}", "legendFormat": "OSPF Admin", "instant": true}
        ],
        "options": {
          "reduceOptions": {"calcs": ["lastNotNull"]},
          "colorMode": "background",
          "graphMode": "none"
        },
        "fieldConfig": {
          "defaults": {
            "mappings": [
              {
                "type": "value",
                "options": {
                  "1": {"text": "Enabled", "color": "green", "index": 0},
                  "2": {"text": "Disabled", "color": "red", "index": 1}
                }
              }
            ]
          }
        }
      },
      {
        "id": 10,
        "title": "Interface RX Traffic (bps)",
        "type": "timeseries",
        "gridPos": {"x": 0, "y": 20, "w": 12, "h": 8},
        "datasource": {"type": "prometheus", "uid": "efglbr4gf9b7ka"},
        "targets": [
          {
            "expr": "rate(ifHCInOctets{instance=\"$instance\", ifName=~\"$ifName\"}[5m]) * 8",
            "legendFormat": "{{ifName}}"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "bps",
            "custom": {"lineWidth": 1, "fillOpacity": 5}
          }
        },
        "options": {"tooltip": {"mode": "multi", "sort": "desc"}}
      },
      {
        "id": 11,
        "title": "Interface TX Traffic (bps)",
        "type": "timeseries",
        "gridPos": {"x": 12, "y": 20, "w": 12, "h": 8},
        "datasource": {"type": "prometheus", "uid": "efglbr4gf9b7ka"},
        "targets": [
          {
            "expr": "rate(ifHCOutOctets{instance=\"$instance\", ifName=~\"$ifName\"}[5m]) * 8",
            "legendFormat": "{{ifName}}"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "bps",
            "custom": {"lineWidth": 1, "fillOpacity": 5}
          }
        },
        "options": {"tooltip": {"mode": "multi", "sort": "desc"}}
      },
      {
        "id": 12,
        "title": "Interface RX Packets/s",
        "type": "timeseries",
        "gridPos": {"x": 0, "y": 28, "w": 12, "h": 8},
        "datasource": {"type": "prometheus", "uid": "efglbr4gf9b7ka"},
        "targets": [
          {
            "expr": "rate(ifHCInUcastPkts{instance=\"$instance\", ifName=~\"$ifName\"}[5m])",
            "legendFormat": "{{ifName}} Ucast"
          },
          {
            "expr": "rate(ifHCInMulticastPkts{instance=\"$instance\", ifName=~\"$ifName\"}[5m])",
            "legendFormat": "{{ifName}} Mcast"
          },
          {
            "expr": "rate(ifHCInBroadcastPkts{instance=\"$instance\", ifName=~\"$ifName\"}[5m])",
            "legendFormat": "{{ifName}} Bcast"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "pps",
            "custom": {"lineWidth": 1, "fillOpacity": 5}
          }
        }
      },
      {
        "id": 13,
        "title": "Interface TX Packets/s",
        "type": "timeseries",
        "gridPos": {"x": 12, "y": 28, "w": 12, "h": 8},
        "datasource": {"type": "prometheus", "uid": "efglbr4gf9b7ka"},
        "targets": [
          {
            "expr": "rate(ifHCOutUcastPkts{instance=\"$instance\", ifName=~\"$ifName\"}[5m])",
            "legendFormat": "{{ifName}} Ucast"
          },
          {
            "expr": "rate(ifHCOutMulticastPkts{instance=\"$instance\", ifName=~\"$ifName\"}[5m])",
            "legendFormat": "{{ifName}} Mcast"
          },
          {
            "expr": "rate(ifHCOutBroadcastPkts{instance=\"$instance\", ifName=~\"$ifName\"}[5m])",
            "legendFormat": "{{ifName}} Bcast"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "pps",
            "custom": {"lineWidth": 1, "fillOpacity": 5}
          }
        }
      }
    ]
  },
  "folderId": 0,
  "overwrite": true
}

```
<!-- DASHBOARD_JSON_END -->

---

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



