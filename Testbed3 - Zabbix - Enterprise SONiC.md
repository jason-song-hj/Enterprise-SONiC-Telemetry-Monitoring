

#### Micas W6510-32C Switch m65_1$ config SNMP ####
 By default, No SNMP start. So you need to config it first.
```bash
 xxx$ sonic-cli
 sonic# configure t
 sonic(config)# snmp-server community public
```

#### Import Enterprise SONiC 4.5.2 template Zabbix Server ####
- Data Collection - Templates - Import - Please Download this and import.<br>
[Enterprise_SONiC_4.5.2_export_templates](https://github.com/qwert22356/Enterprise-SONiC-Telemetry-Monitoring-Testbed/blob/main/Zabbix%20Resources/Enterprise_SONiC_4.5.2_export_templates.yaml)

Enterprise SONiC 239 items which were like CPU/Mem/Fan/PSU/Ports/DDM/ARP/MAC/v4/6_entries/OSFP/BGP etc...  

<img width="1725" height="294" alt="image" src="https://github.com/user-attachments/assets/e91f7e1e-5b9a-42b9-94d6-5c6ff708b3fc" />

---

<img width="1875" height="826" alt="image" src="https://github.com/user-attachments/assets/67fd4421-edf2-47d4-905d-45421925d509" />

---

#### Zabbix Server - Create Dashboard for SONiC Switches ####

- First - Go to Monitoring Hosts to create new Host 
<img width="1913" height="884" alt="image" src="https://github.com/user-attachments/assets/9794a0e3-2617-4b3f-a66f-8c87c97328d4" />
---

- Second - Directly use the Promote let LLM to help you Faster Creating a new Dashboard for your Host that you created ...

Before you put promote to LLM, You need to Download [JSON file](https://github.com/qwert22356/Enterprise-SONiC-Telemetry-Monitoring-Testbed/blob/main/Zabbix%20Resources/sonic_dashboard_examples.json)

```bash
Example Promote - Claude 
I have a Zabbix Dashboard JSON backup file, and I need you to help me rebuild it on a new Zabbix instance.

Important requirements:
1 The itemid/graphid in the JSON are internal IDs of the old machine. On the new machine, these IDs have already changed and cannot be used directly.
2 You need to first dynamically query through the Zabbix API, using item key, graph name, and host name to find the corresponding real IDs on the new machine.
3 Generate a Python script, the workflow is as follows:
- First call user.login to get the token (in Zabbix 7.x, the returned result string is the token)
- Put the token in the header of all subsequent requests: Authorization: Bearer <token>, do not put it in the auth field of the request body
- Query hostid based on host name
- Iterate through each page and each widget of the Dashboard
- For graph widget: use graph name to query the new graphid
- For item widget: use item key to query the new itemid
- For honeycomb widget: rebuild using item pattern
- Use the retrieved new IDs to rebuild the entire Dashboard
4 Zabbix version is <x.x>, Dashboard total width is 72 columns
5 Zabbix URL: <http://100.88.xx.xx:8080>, username <Admin>, password <admin>, host name <Put your Hosts - name here> 

JSON file as follows:
[JSON file]
```

---

After this, LLM will give you python script and you can run it on your Zabbix Server...

#### Zabbix Server - This is what it would be looks like ####

<img width="1893" height="892" alt="image" src="https://github.com/user-attachments/assets/656ceee1-edbd-4837-8a5d-69d9b7d66ac8" />
<img width="1897" height="893" alt="image" src="https://github.com/user-attachments/assets/3818e526-d9a3-4883-9c50-56be82fbf50c" />
<img width="1894" height="894" alt="image" src="https://github.com/user-attachments/assets/f3622493-8cd3-4ede-b184-50816037ba4e" />
<img width="1889" height="892" alt="image" src="https://github.com/user-attachments/assets/6d495bfd-39cc-4a9c-a615-f33f5e638e9f" />

---





  

