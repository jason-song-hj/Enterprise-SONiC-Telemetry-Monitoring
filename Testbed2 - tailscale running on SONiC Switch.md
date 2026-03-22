<p align="center">
  <i>Testbed2: Tailscale Test on Enterprise SONiC (Broadcom) Platform</i>
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

## 📌 Background

In many deployment scenarios, SONiC switches are located in remote environments or customer sites where direct access is limited.

Traditional approaches (VPN / jump server / bastion host) can be:

- Complex to deploy
- Hard to maintain for small-scale environments
- Not flexible for quick testing or troubleshooting

This test demonstrates a lightweight approach:

👉 Using **Tailscale** to securely connect SONiC switches into a private network  
👉 Enabling remote access **without complex network configuration**

---

## 🎯 Objective

- Enable secure remote access to SONiC switches from anywhere
- Simplify connectivity without requiring VPN or firewall changes
- Validate Tailscale deployment directly on SONiC (via script)
- Demonstrate real-world operability for remote troubleshooting and demos

---

## 🎥 Demo Overview

This demo shows:

1. Generate Tailscale install script
2. Install and enable Tailscale on SONiC
3. Register switch into private network
4. Access SONiC remotely from external device

---

## ⚙️ Steps

### Step 1 - Generate Tailscale install script

<img src="https://github.com/user-attachments/assets/644c05f5-0a45-476c-8a17-997fc9d53a82" width="800"/>

---

<img src="https://github.com/user-attachments/assets/0284a4d1-6dc6-4278-8c61-80ec54f7bd4d" width="600"/>

---

### Step 2 - Login SONiC and paste script

<img src="https://github.com/user-attachments/assets/101e4822-06d0-4925-a11f-5e0a35f0e6ca" width="800"/>

---

<img src="https://github.com/user-attachments/assets/e2b55edd-8f5b-4039-a20e-a3970a97144d" width="600"/>

---

### Step 3 - Enable Tailscale

```bash
sudo tailscale up
```

### Step 4 - See your SONiC sw on tailscale 

<img width="1189" height="63" alt="image" src="https://github.com/user-attachments/assets/e09dc1a0-9551-4adb-a57c-84352d150b0e" />

---

### Step 5 - Test tailscale from anywhere

<img width="859" height="451" alt="image" src="https://github.com/user-attachments/assets/b027e245-93dc-471c-9d23-ce32e0530a31" />


---

