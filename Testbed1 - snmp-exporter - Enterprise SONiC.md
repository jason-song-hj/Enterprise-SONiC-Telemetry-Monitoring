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
  <b>Lightweight monitoring SNMP-Exporter_verification for small-scale SONiC deployments</b>
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

## DEMO & Results
#### On Micas W6510-32C Switch + m65_1$ new folder for SNMP-Exporter ####
```bash
mkdir /opt/snmp-exporter
```
```bash
sudo su
cd /opt/snmp-exporter
vi snmp.yml
```
- Insert this case for simple test
```bash
auths:
  public_v2:
    version: 2
    community: public

modules:
  sonic_if:
    walk:
      - 1.3.6.1.2.1.2
      - 1.3.6.1.2.1.31

    metrics:

      - name: ifName
        oid: 1.3.6.1.2.1.31.1.1.1.1
        type: DisplayString
        indexes:
          - labelname: ifIndex
            type: gauge

      - name: ifOperStatus
        oid: 1.3.6.1.2.1.2.2.1.8
        type: gauge
        indexes:
          - labelname: ifIndex
            type: gauge

      - name: ifHCInOctets
        oid: 1.3.6.1.2.1.31.1.1.1.6
        type: counter
        indexes:
          - labelname: ifIndex
            type: gauge
        lookups:
          - labels: [ifIndex]
            labelname: ifName
            oid: 1.3.6.1.2.1.31.1.1.1.1

      - name: ifHCOutOctets
        oid: 1.3.6.1.2.1.31.1.1.1.10
        type: counter
        indexes:
          - labelname: ifIndex
            type: gauge
```

#### On Micas W6510-32C Switch m65_1$ ####
```bash
docker run -d \
 --name snmp-exporter \
 -p 9116:9116 \
 -v /opt/snmp-exporter/snmp.yml:/etc/snmp_exporter/snmp.yml \
 prom/snmp-exporter
```

 #### On Micas W6510-32C Switch m65_1$ config SNMP ####
 By default, No SNMP start. So you need to config it first.
```bash
 xxx$ sonic-cli
 sonic# configure t
 sonic(config)# snmp-server community public
```

 #### On Micas W6510-32C Switch m65_1$ test SNMP-exporter ####
```bash
 curl "http://192.168.1.124:9116/snmp?target=192.168.1.124&module=sonic_if"
```
 admin@sonic:~$ curl "http://192.168.1.124:9116/snmp?target=192.168.1.124&module=sonic_if"
```bash
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

#### Remote PC test Resources of Switches test ####

You can see the docker status snmp-exporter system performance and resources
<img width="898" height="45" alt="image" src="https://github.com/user-attachments/assets/0ffe4ebe-b3e3-484b-9fa9-cc271ff9572f" />

when you run snmp walk  snmpwalk -v2c -c public 192.168.xxx.x
<img width="1329" height="903" alt="image" src="https://github.com/user-attachments/assets/6fe82728-9b85-4226-91f5-62364e1ff3b0" />
<img width="1329" height="51" alt="image" src="https://github.com/user-attachments/assets/e2542dc8-fd63-472f-9163-c4eb0fd0b0b0" />
</br>
Performance CPU 0.00%, MEM 0.6% so it was not very occupied and just little resources needed of MEM.


#### LoginServer - Config Prometheus ####
```bash
scrape_configs:
  - job_name: 'sonic_snmp'

    metrics_path: /snmp

    params:
      module: [sonic_if]   # 👈 snmp.yml module
      auth: [public_v2]

    static_configs:
      - targets:
        - 192.168.1.124   # 👈 SONiC IP

    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target

      - source_labels: [__param_target]
        target_label: instance

      - target_label: __address__
        replacement: 192.168.1.124:9116   # 👈 snmp-exporter address
```
- you must create a prometheus.yml file in your prometheus folder
- This is my pwd:  /home/cat/datahub/prometheus/prometheus.yml
  
```bash
docker restart prometheus
```
---
- Go to your prometheus address: 192.168.x.x:9090/targets
<img width="1899" height="294" alt="image" src="https://github.com/user-attachments/assets/ba7c44d3-cc94-4715-bda2-5cc60fd11577" />
- It shows up for this Endpoint

#### LoginServer - Config Grafana ####
- Go to Grafana IP: 192.168.x:3000 - connections - add new conn
- Search Prometheus - click Prometheus - add new datasource - Put your prometheus ip add on the address
- Go to Dashboard - new dashboard - import dashboardjson * for an example:
```bash
{
  "annotations": {
    "list": [
      {
        "builtIn": 1,
        "datasource": {
          "type": "grafana",
          "uid": "-- Grafana --"
        },
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "name": "Annotations & Alerts",
        "type": "dashboard"
      }
    ]
  },
  "editable": true,
  "fiscalYearStartMonth": 0,
  "graphTooltip": 0,
  "links": [],
  "panels": [
    {
      "fieldConfig": {
        "defaults": {
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": 0
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "bps"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 4,
        "w": 6,
        "x": 0,
        "y": 0
      },
      "id": 1,
      "options": {
        "colorMode": "value",
        "graphMode": "area",
        "justifyMode": "auto",
        "orientation": "auto",
        "percentChangeColorMode": "standard",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "showPercentChange": false,
        "textMode": "auto",
        "wideLayout": true
      },
      "pluginVersion": "12.4.1",
      "targets": [
        {
          "expr": "sum(rate(ifHCInOctets[1m]) * 8)",
          "refId": "A"
        }
      ],
      "title": "Total Ingress Traffic",
      "type": "stat"
    },
    {
      "fieldConfig": {
        "defaults": {
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": 0
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "bps"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 4,
        "w": 6,
        "x": 6,
        "y": 0
      },
      "id": 2,
      "options": {
        "colorMode": "value",
        "graphMode": "area",
        "justifyMode": "auto",
        "orientation": "auto",
        "percentChangeColorMode": "standard",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "showPercentChange": false,
        "textMode": "auto",
        "wideLayout": true
      },
      "pluginVersion": "12.4.1",
      "targets": [
        {
          "expr": "sum(rate(ifHCOutOctets[1m]) * 8)",
          "refId": "A"
        }
      ],
      "title": "Total Egress Traffic",
      "type": "stat"
    },
    {
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisBorderShow": false,
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "barWidthFactor": 0.6,
            "drawStyle": "line",
            "fillOpacity": 0,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "insertNulls": false,
            "lineInterpolation": "linear",
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "auto",
            "showValues": false,
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": 0
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "bps"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 24,
        "x": 0,
        "y": 4
      },
      "id": 3,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "hideZeros": false,
          "mode": "single",
          "sort": "none"
        }
      },
      "pluginVersion": "12.4.1",
      "targets": [
        {
          "expr": "rate(ifHCInOctets[1m]) * 8",
          "legendFormat": "RX {{ifIndex}}",
          "refId": "A"
        },
        {
          "expr": "rate(ifHCOutOctets[1m]) * 8",
          "legendFormat": "TX {{ifIndex}}",
          "refId": "B"
        }
      ],
      "title": "Interface Traffic (Per Port)",
      "type": "timeseries"
    },
    {
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisBorderShow": false,
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "barWidthFactor": 0.6,
            "drawStyle": "line",
            "fillOpacity": 0,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "insertNulls": false,
            "lineInterpolation": "linear",
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "auto",
            "showValues": false,
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": 0
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "percent"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 24,
        "x": 0,
        "y": 12
      },
      "id": 4,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "hideZeros": false,
          "mode": "single",
          "sort": "none"
        }
      },
      "pluginVersion": "12.4.1",
      "targets": [
        {
          "expr": "(rate(ifHCInOctets[1m]) * 8) / 100000000000 * 100",
          "legendFormat": "RX {{ifIndex}}",
          "refId": "A"
        },
        {
          "expr": "(rate(ifHCOutOctets[1m]) * 8) / 100000000000 * 100",
          "legendFormat": "TX {{ifIndex}}",
          "refId": "B"
        }
      ],
      "title": "Interface Utilization (%)",
      "type": "timeseries"
    },
    {
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisBorderShow": false,
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "fillOpacity": 80,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineWidth": 1,
            "scaleDistribution": {
              "type": "linear"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": 0
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "bps"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 0,
        "y": 20
      },
      "id": 5,
      "options": {
        "barRadius": 0,
        "barWidth": 0.97,
        "fullHighlight": false,
        "groupWidth": 0.7,
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "orientation": "auto",
        "showValue": "auto",
        "stacking": "none",
        "tooltip": {
          "hideZeros": false,
          "mode": "single",
          "sort": "none"
        },
        "xTickLabelRotation": 0,
        "xTickLabelSpacing": 0
      },
      "pluginVersion": "12.4.1",
      "targets": [
        {
          "expr": "topk(10, rate(ifHCInOctets[1m]) * 8)",
          "legendFormat": "{{ifIndex}}",
          "refId": "A"
        }
      ],
      "title": "Top 10 Interfaces (Ingress)",
      "type": "barchart"
    },
    {
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisBorderShow": false,
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "fillOpacity": 80,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineWidth": 1,
            "scaleDistribution": {
              "type": "linear"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": 0
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "bps"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 12,
        "y": 20
      },
      "id": 6,
      "options": {
        "barRadius": 0,
        "barWidth": 0.97,
        "fullHighlight": false,
        "groupWidth": 0.7,
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "orientation": "auto",
        "showValue": "auto",
        "stacking": "none",
        "tooltip": {
          "hideZeros": false,
          "mode": "single",
          "sort": "none"
        },
        "xTickLabelRotation": 0,
        "xTickLabelSpacing": 0
      },
      "pluginVersion": "12.4.1",
      "targets": [
        {
          "expr": "topk(10, rate(ifHCOutOctets[1m]) * 8)",
          "legendFormat": "{{ifIndex}}",
          "refId": "A"
        }
      ],
      "title": "Top 10 Interfaces (Egress)",
      "type": "barchart"
    },
    {
      "fieldConfig": {
        "defaults": {
          "mappings": [
            {
              "options": {
                "1": {
                  "color": "green",
                  "text": "UP"
                },
                "2": {
                  "color": "red",
                  "text": "DOWN"
                }
              },
              "type": "value"
            }
          ],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": 0
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 4,
        "w": 24,
        "x": 0,
        "y": 28
      },
      "id": 7,
      "options": {
        "colorMode": "value",
        "graphMode": "area",
        "justifyMode": "auto",
        "orientation": "auto",
        "percentChangeColorMode": "standard",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "showPercentChange": false,
        "textMode": "auto",
        "wideLayout": true
      },
      "pluginVersion": "12.4.1",
      "targets": [
        {
          "expr": "ifOperStatus",
          "refId": "A"
        }
      ],
      "title": "Interface Status",
      "type": "stat"
    }
  ],
  "preload": false,
  "refresh": "",
  "schemaVersion": 42,
  "tags": [],
  "templating": {
    "list": []
  },
  "time": {
    "from": "now-6h",
    "to": "now"
  },
  "timepicker": {},
  "timezone": "browser",
  "title": "SONiC Interface Monitoring (100G)",
  "uid": "adsmqhv",
  "version": 1,
  "weekStart": ""
}
```
---
<img width="1575" height="801" alt="image" src="https://github.com/user-attachments/assets/5662b95e-a3f4-45b6-9b80-131e2b9ef6fb" />

- This is just a demo for dashboard





