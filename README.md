<h1 align="center">SONiC Observability Lab</h1>

<p align="center">
  <i>Enterprise SONiC Telemetry & Monitoring Testbed</i>
</p>

<p align="center">
  Built by <b>Jason S</b>
</p>

<!-- 
<p align="center">
  <a href="https://youtube.com/你的频道">
    <img src="https://img.shields.io/badge/▶ Watch Demo-YouTube-red?style=for-the-badge&logo=youtube"/>
  </a>
</p>
-->

---

## 🚀 What This Repo Shows

This repository demonstrates real-world observability capabilities on **Broadcom SONiC**, including:

- 📊 SNMP monitoring (Prometheus + snmp-exporter)
- 📡 gNMI telemetry streaming
- ⚡ gRPC-based data collection
- 🔍 Interface / traffic / error analysis
- 🧪 Lab-based reproducible test cases

---

## 🧪 Test Scenarios & Roadmap

Testing is progressing step by step. The 2026 roadmap focuses on mainstream DevOps ecosystems and advanced Telemetry features, rolling out new scenarios sequentially.


| Module | Description | Status / Demo |
|--------|------------|------|
| **Tailscale** | Secure mesh VPN networking & node access | [View](./tailscale/README.md) |
| **Zabbix** | Infrastructure monitoring & alerting | [View](./zabbix/README.md) |
| **Prometheus** | Time-series data & metrics collection | [View](./prometheus/README.md) |
| **Ansible** | Automated provisioning & configuration | [View](./ansible/README.md) |

---

### 🚀 2026 Advanced Testing Roadmap (Continuously Updated)

<details>
<summary>🗺️ <b>Click to expand Advanced Telemetry & Third-Party Integration Plans</b></summary>
<br>

#### 1. Advanced SONiC Telemetry Insights
- [ ] **MOD Packet Loss Visualization**: Real-time mirroring and precise loss tracking at the NIC/switch level.
- [ ] **LDH Path Latency Visualization**: Network-wide latency probing and dynamic path topology rendering.

#### 2. Third-Party Tools & AI-Driven NetOps
- [ ] **AI-Powered Automation**: Leveraging **Claude Code** for deep network operations and automated troubleshooting.
- [ ] **Architecture Evolution**: Implementing **Hermes** NetOps solutions and deployment best practices.

</details>

---

## 🎥 Demo Videos & Community

> 💡 **What would you like to see next?**  
> More test scenarios and video demos are currently in production! If you have specific ideas regarding **MOD packet loss** or **Claude Code operations**, head over to my **[YouTube Channel (@jason_hj)](https://youtube.com)**. Subscribe and drop a comment—I will prioritize topics based on your feedback!

---

## 🧠 Why This Matters

Most SONiC deployments lack a **unified observability layer**.

This repo shows how to build with 3rd tools step by step.
