<h1 align="center">SONiC Observability Lab</h1>

<p align="center">
  <i>Enterprise SONiC Telemetry & Monitoring Testbed</i>
</p>

<p align="center">
  Built by <b>Jason S</b>
</p>

<p align="center">
  <a href="https://youtube.com/你的频道">
    <img src="https://img.shields.io/badge/▶ Watch Demo-YouTube-red?style=for-the-badge&logo=youtube"/>
  </a>
</p>

---

## 🚀 What This Repo Shows

This repository demonstrates real-world observability capabilities on **Broadcom SONiC**, including:

- 📊 SNMP monitoring (Prometheus + snmp-exporter)
- 📡 gNMI telemetry streaming
- ⚡ gRPC-based data collection
- 🔍 Interface / traffic / error analysis
- 🧪 Lab-based reproducible test cases

---

## 🧪 Test Scenarios

| Module | Description | Demo |
|--------|------------|------|
| SNMP Monitoring | Interface + counters via SNMP exporter | [View](./snmp/README.md) |
| gNMI Telemetry | Streaming telemetry via gnmic | [View](./gnmi/README.md) |
| gRPC Collector | gRPC-based data ingestion | [View](./grpc/README.md) |

---

## 🎥 Demo Videos

- SNMP Demo → https://youtube.com/xxx
- gNMI Demo → https://youtube.com/xxx

---

## 🧠 Why This Matters

Most SONiC deployments lack a **unified observability layer**.

This repo shows how to build:

```text
SNMP + gNMI + Streaming → Unified DataHub
