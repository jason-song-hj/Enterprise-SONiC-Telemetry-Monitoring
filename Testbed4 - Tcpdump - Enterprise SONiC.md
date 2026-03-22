<p align="center">
  <i>Testbed4: tcpdump Live Packet Capture on Enterprise SONiC (Broadcom) Platform</i>
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
  <b>Agentless packet capture for real-time traffic analysis on SONiC — no SPAN, no Wireshark laptop required</b>
</p>

## 📌 Background

In traditional network environments, capturing packets requires physical access and dedicated hardware:

- Configure a **SPAN/mirror port** on the switch
- Connect a laptop with Wireshark to the mirror port
- Analyze traffic locally — impossible to do remotely

This approach is impractical for:
- Remote or unmanned sites
- Cloud/lab environments without physical access
- Fast troubleshooting during incidents

SONiC runs on **standard Linux**, which means `tcpdump` is available natively in the shell.  
This unlocks a fundamentally different workflow:

👉 SSH into the switch directly (including via **Tailscale** for secure out-of-band access)  
👉 Run `sudo tcpdump` on any interface — Management, front-panel, loopback  
👉 Capture **TCP, UDP, ICMP, ARP, BGP, OSPF, VXLAN** and more — all from one tool  
👉 Save `.pcap` files for offline Wireshark analysis if needed

## 🎯 Objective

- Validate tcpdump availability and usability on Enterprise SONiC
- Demonstrate real-world capture scenarios (control plane, management traffic, overlay)
- Show how Tailscale + tcpdump enables secure remote packet capture without SPAN
- Compare workflow efficiency vs traditional mirror-based capture

## 🧪 What This Test Covers

- tcpdump on SONiC Linux shell (interface naming, permissions, sudo behavior)
- Management plane traffic capture (SSH, Tailscale DERP vs direct peer)
- Control plane protocol capture (BGP, OSPF, ARP)
- Checksum offload behavior (why `incorrect` checksum is normal)
- Saving and transferring `.pcap` files for Wireshark analysis
- Resource impact of running tcpdump on-box
