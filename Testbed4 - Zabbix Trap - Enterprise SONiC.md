<p align="center">
  <i>Enterprise SONiC SNMP Trap → Zabbix Integration</i>
</p>

<p align="center">
  Built by <b>Jason S</b>
</p>

<p align="center">
  <a href="https://www.youtube.com/watch?v=yZ-0n-A-ZsM">
    <img src="https://img.shields.io/badge/▶ Watch Demo-YouTube-red?style=for-the-badge&logo=youtube"/>
  </a>
</p>

---

<p align="center">
  <b>Receive SONiC SNMP Traps and display in Zabbix Dashboard</b>
</p>

---

## 🏗️ Architecture

```
SONiC Switch
  └── SNMP Trap (UDP 162)
        └──► Zabbix host machine :1162 (via iptables redirect)
                └── zabbix-snmptraps (Docker) → ./snmptraps/snmptraps.log
                        └──► zabbix-server (Docker, read-only volume)
                                └──► Dashboard
```

---

## 1️⃣ SONiC Switch — SNMP Trap Configuration

```bash
sonic# configure
sonic(config)# snmp-server host <ZABBIX_HOST_IP> trap version 2c public
sonic(config)# exit

# Verify
sonic# show snmp-server host
```

---

## 2️⃣ Zabbix Server — SNMP Trap Configuration

### 2.1 iptables — Redirect UDP 162 → 1162

The `zabbix-snmptraps` container listens on port **1162** internally. Redirect incoming traps from the standard port 162:

```bash
sudo iptables -t nat -A PREROUTING -p udp --dport 162 -j REDIRECT --to-port 1162
```

### 2.2 Create snmptraps folder

```bash
cd <your_datahub_folder>
mkdir -p snmptraps
```

> ⚠️ The `snmptraps` folder must be in the same directory as your `docker-compose.yml`

### 2.3 docker-compose.yml

```yaml
version: "3.8"
services:
  zabbix-server:
    image: zabbix/zabbix-server-pgsql:latest
    container_name: zabbix-server
    restart: always
    environment:
      DB_SERVER_HOST: zabbix-db
      POSTGRES_USER: zabbix
      POSTGRES_PASSWORD: zabbix
      POSTGRES_DB: zabbix
      ZBX_ENABLE_SNMP_TRAPS: "true"
    depends_on:
      - zabbix-db
      - zabbix-snmptraps
    ports:
      - "10051:10051"
    volumes:
      - ./snmptraps:/var/lib/zabbix/snmptraps:ro
    networks:
      - datahub-net

  zabbix-web:
    image: zabbix/zabbix-web-nginx-pgsql:latest
    container_name: zabbix-web
    restart: always
    environment:
      DB_SERVER_HOST: zabbix-db
      POSTGRES_USER: zabbix
      POSTGRES_PASSWORD: zabbix
      POSTGRES_DB: zabbix
      ZBX_SERVER_HOST: zabbix-server
      PHP_TZ: Europe/Madrid
    ports:
      - "8080:8080"
    depends_on:
      - zabbix-server
    networks:
      - datahub-net

  zabbix-db:
    image: postgres:15
    container_name: zabbix-db
    restart: always
    environment:
      POSTGRES_DB: zabbix
      POSTGRES_USER: zabbix
      POSTGRES_PASSWORD: zabbix
    volumes:
      - ./zabbix-db:/var/lib/postgresql/data
    networks:
      - datahub-net

  zabbix-snmptraps:
    image: zabbix/zabbix-snmptraps:latest
    container_name: zabbix-snmptraps
    restart: always
    network_mode: "host"
    volumes:
      - ./snmptraps:/var/lib/zabbix/snmptraps

networks:
  datahub-net:
    driver: bridge
    ipam:
      config:
        - subnet: 10.30.0.0/16
```

> You can customize the `docker-compose.yml` to fit your environment

```bash
docker compose up -d

# Verify
docker ps | grep zabbix
```

---

## 3️⃣ Zabbix UI Configuration

### 3.1 Configure Host SNMP Interface

`Data collection → Hosts → <your host> → Interfaces`

Add an **SNMP** interface with the SONiC switch IP address.

### 3.2 Create SNMP Trap Item

`Data collection → Hosts → <hostname> → Items → Create item`

| Field | Value |
|-------|-------|
| Name | `SNMP Trap` |
| Type | `SNMP trap` |
| Key | `snmptrap.fallback` |
| Type of information | `Log` |

---

## 4️⃣ Test & Verify

### Simulate a trap on SONiC

```bash
# Restart SNMP service on SONiC to trigger a Cold Start trap
admin$ docker snmp restart
```

### Watch the trap log on host machine

```bash
Zabbix$ tail -f ~/datahub/snmptraps/snmptraps.log
```

A new entry should appear:

<img width="598" height="65" alt="snmptraps log" src="https://github.com/user-attachments/assets/16a297eb-7a52-4991-aa83-f562d402ee93" />

---

## 5️⃣ Dashboard Setup

### Check Latest Data

`Monitoring → Latest data` — filter by host to see incoming traps:

<img width="1903" height="866" alt="Latest Data" src="https://github.com/user-attachments/assets/e6c83806-d91a-4e9e-a842-8760eaa5b48f" />

---

### Add History Widget to Dashboard

`Monitoring → Dashboard → Edit dashboard → Add widget → History`

Select the trap items you want to display:

<img width="1012" height="459" alt="Add Widget" src="https://github.com/user-attachments/assets/4eb2a322-f621-44b9-9bd7-6ba3ee0a009a" />

---

### Final Dashboard

<img width="1883" height="664" alt="Dashboard" src="https://github.com/user-attachments/assets/40542070-c96f-4e5c-b9d2-0e82273c3ee6" />

---

## 🔍 Troubleshooting

```bash
# Watch trap log in real time
tail -f <your_datahub_folder>/snmptraps/snmptraps.log

# Check zabbix-snmptraps container logs
docker logs zabbix-snmptraps | tail -30

# Verify iptables redirect is active
sudo iptables -t nat -L PREROUTING -n | grep 162

# Capture packets to verify SONiC is sending traps
tcpdump -i any udp port 162 -n
```

---

## 📝 Notes

- `zabbix-snmptraps` uses `network_mode: "host"` — do **not** add a `ports:` section alongside it
- The `snmptraps` volume is mounted **read-only** (`ro`) in `zabbix-server` and **read-write** in `zabbix-snmptraps`
- Use `snmptrap.fallback` as the item key to catch all incoming traps; use `snmptrap[<regex>]` to filter specific trap types
- Trap messages include OID, source IP, and trap content — useful for building triggers based on specific events




