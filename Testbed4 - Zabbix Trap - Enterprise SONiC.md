#### Config SNMP Trap on Ent_SONiC ####

snmp-server host 100.105.8.x trap 

show snmp-server host 


#### Docker-Compose file ####
```bash
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
    ports: 
      - "162:1162/udp"
    volumes:
      - ./snmptraps:/var/lib/zabbix/snmptraps

networks:
  datahub-net:
    driver: bridge
    ipam:
      config:
        - subnet: 10.30.0.0/16
```
You can change the way that your docker-compose yml file as you like     

#### Make new folder ####
cd your_target_foder
mkdir -p snmptraps //keep same docker-composepwd

#### Tranfer Iptables 162 -> 1162 ####
sudo iptables -t nat -A PREROUTING -p udp --dport 162 -j REDIRECT --to-port 1162

#### Test on Rk3588 ####
tail -f ~/datahub/snmptraps/snmptraps.log 

#### Simulate a trap on SONiC ####
docker snmp restart

you will see a new item appears at Zabbix Server
<br>
<img width="598" height="65" alt="image" src="https://github.com/user-attachments/assets/16a297eb-7a52-4991-aa83-f562d402ee93" />

#### Go to your Zabbix Server Dashboard ####
Check Monitoring - latest data to see trap
<img width="1903" height="866" alt="image" src="https://github.com/user-attachments/assets/e6c83806-d91a-4e9e-a842-8760eaa5b48f" />

---

Dashboard - add wid - item history - you can add the items trap that you want.
<img width="1012" height="459" alt="image" src="https://github.com/user-attachments/assets/4eb2a322-f621-44b9-9bd7-6ba3ee0a009a" />

---

Finnally My Dashboard
<img width="1883" height="664" alt="image" src="https://github.com/user-attachments/assets/40542070-c96f-4e5c-b9d2-0e82273c3ee6" />



