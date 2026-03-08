# ⚓ Nordhamn Oil Transfer (OT) Simulator

<div align="center">
  <img src="https://img.shields.io/badge/Status-Development-orange?style=for-the-badge" alt="Status">
  <img src="https://img.shields.io/badge/Platform-Raspberry%20Pi-C51A4A?style=for-the-badge&logo=raspberry-pi" alt="Raspberry Pi">
  <img src="https://img.shields.io/badge/Docker-Enabled-2496ED?style=for-the-badge&logo=docker" alt="Docker">
  <img src="https://img.shields.io/badge/Grafana-Visualization-F46800?style=for-the-badge&logo=grafana" alt="Grafana">
  <img src="https://img.shields.io/badge/Suricata-IDS/IPS-EF3B2D?style=for-the-badge&logo=linux" alt="Suricata">
</div>

---

## 📖 Om Projektet
Detta projekt fokuserar på att emulera de industriella styrsystemen (OT) vid en simulerad hamn "Nordhamn" och dess Oljeterminal, Kaj 8314. Syftet är att skapa en höginteraktiv honeypot som simulerar en Siemens S7-PLC för att studera exponering av kritiska protokoll och nätverksinteraktioner i en kontrollerad miljö. Projektet är designat som en modern Security Operations Center (SOC)-pipeline, där sensorer samlar in data som sedan övervakas, visualiseras och analyseras.

## 🧠 Metodik & AI som "Force Multiplier"
Min kärnkompetens i detta projekt ligger på den teoretiska förståelsen för OT-säkerhet – arkitektur, nätverkssegmentering, hotmodellering och driftsäkerhet.

För att översätta denna teoretiska kunskap till praktiskt utförande (avancerad Linux-härdning, reverse-engineering av XML-scheman och Docker-orkestrering) har jag aktivt använt LLM-verktyg som en teknisk sparringpartner. Detta demonstrerar inte bara implementering av OT-säkerhetsprinciper i praktiken, utan bevisar också förmågan att agilt använda modern AI för att snabbt och säkert realisera komplexa säkerhetsarkitekturer.

## 🛠️ Teknisk Stack
* **Simulator:** Conpot v0.6.0 (MushMush Foundation)
* **Monitoring & IDS:** Suricata, Promtail & Loki *(Kommande)*
* **Visualisering:** Grafana *(Kommande)*
* **Infrastruktur:** Docker & Docker Compose
* **Hårdvara:** Raspberry Pi 5 med NVMe-lagring (Sentry-nod)
* **Protokoll:** S7Comm (102), Modbus (502), HTTP (80/8080)

---

## 🗺️ Roadmap: Bygga en Automatiserad SOC
Denna honeypot är fundamentet i en större Threat Intelligence-pipeline.

- [x] **Fas 1: Core Architecture & Frontend** (Custom XML, Web UI, Docker, Edge-konfiguration)
- [x] **Fas 2: Protokoll-Simulering** (Konfigurering av realistiska Modbus-register och S7-noder för "Pump Station 01")
- [ ] **Fas 3: Nätverksövervakning & Dashboard** (Suricata IDS & Grafana)
- [ ] **Fas 4: Automatisering av Threat Intel** (Integration av n8n-workflows)
- [ ] **Fas 5: AI-Analys** (AI-agenter som tolkar råa loggfiler och identifierar attackmönster)
- [ ] **Fas 6: Rapportering** (Automatisk export av berikad attackdata till Excel)



1. Arkitektur och Miljöoptimering
Skapat en robust projektstruktur och åtgärdat kritiska stabilitetsproblem i simulatorns Virtual File System (VFS).

Fix för ResourceReadOnly: Genom att styra om data_fs_url till mem:// möjliggjordes skrivning i minnet, vilket löste krascher när protokoll (som FTP/TFTP) försökte skriva till låsta filsystem.

Bash
# Skapa logg-miljö
mkdir ~/nordhamn-ot && cd ~/nordhamn-ot
mkdir conpot_logs
2. Deep Dive: Konfigurationsvalidering (conpot.cfg)
Löste "Whack-a-mole"-problematiken i Conpot 0.6.0 där programmet kraschade p.g.a. saknade eller felnamnade sektioner i konfigurationen.

JSON-fix: Tvingade simulatorn att acceptera loggformatet genom att mappa om [json_logger] till [json].

Public IP Parsing: Konfigurerat korrekt array-parsing för externa IP-anrop.

Service Stabilization: Inaktiverat trasiga moduler (Taxii, HPfriends) som annars hindrade uppstart.

3. Spoofing av Identitet (Siemens S7-300)
För att öka trovärdigheten mot angripare har enheten konfigurerats att identifiera sig som en Siemens CPU 315-2, en vanlig arbetshäst inom industriell automation.

XML
<s7comm>
    <SystemDescription>Nordhamn Pump Station Controller PS-01</SystemDescription>
    <OrderCode>6ES7 315-2AG10-0AB0</OrderCode>
    <ModuleType>CPU 315-2 DP</ModuleType>
</s7comm>
4. Modbus Register-mappning
Definierat specifika register för att simulera en levande process vid oljeterminalen:

Register 1001: Vattennivå (0-100%)

Register 1002: Pumptryck (Bar)

Register 1003: Pumpstatus (On/Off)

5. Web Portal Injection (Port 80)
Då Conpots inbyggda mall-system för variabler (<condata>) visade sig vara inkompatibelt med version 0.6.0, implementerades en strategisk override. Genom att injicera en hårdkodad HTML-portal direkt i containerns filsystem via Docker Volumes skapades en trovärdig inloggningssida för Nordhamn.

6. Orchestration med Docker Compose
Använt Bind Mounts för att utföra "kirurgiska ingrepp" i containern utan att behöva bygga om hela imagen vid varje ändring.

YAML
# docker-compose.yml utdrag
volumes:
  - ./conpot.cfg:/etc/conpot/conpot.cfg
  - ./nordhamn_working.xml:/usr/local/lib/python3.9/site-packages/conpot/templates/default/template.xml
  - ./nordhamn_index.html:/usr/local/lib/python3.9/site-packages/conpot/templates/default/http/htdocs/index.html
🔍 Verifiering (Success!)
Verifierat att simulatorn svarar korrekt och ljuger för omvärlden på ett trovärdigt sätt:

Nmap SNMP: Identifierar sig som CP 443-1 EX40 (Siemens nätverkskort).

Nmap S7: Svarar på port 102 som en aktiv PLC.

HTTP: Visar Nordhamns kontrollpanel istället för en felkod.

## ⚠️ Utmaningar & Lärdomar
Under utvecklingsfasen har stor vikt lagts vid att lösa kompatibilitetsproblem i **Conpot 0.6.0**. Arbetet har gett djupgående kunskaper i:
- **Konfigurationsvalidering:** Felsökning av `configparser` och källkod för att möta strikta krav på sektionsindelning i .cfg-filer.
- **Protocol Handshaking:** Analys av varför vissa Nmap-skript (t.ex. `s7-info`) nekas vid emulering och hur man anpassar svar för att öka trovärdigheten.
- **Data Bus Injection:** Pågående arbete med att synkronisera simulatorns interna variabler med det externa webbgränssnittet.

## 🔧 Installation & Användning
1. Klona repot till din Raspberry Pi:
   ```bash
   git clone [https://github.com/jakobgoffe/nordhamn-ot.git](https://github.com/jakobgoffe/nordhamn-ot.git)


## 1️⃣ Grundinstallation (Raspberry Pi OS)

Steg:
Installera Raspberry Pi OS Lite (64-bit) direkt på NVMe-enheten via Raspberry Pi Imager.

Under "OS Customization", konfigurera:

Hostname: sentry

Aktivera SSH (lösenordsautentisering för initial setup).

Konfigurera nätverk.

Säkerställ att Pi 5:ans EEPROM är uppdaterad och konfigurerad för NVMe-boot.

Installera därefter Docker Engine och ge din användare behörighet:

```bash

curl -fsSL [https://get.docker.com](https://get.docker.com) -o get-docker.sh && sudo sh get-docker.sh

sudo usermod -aG docker $USER

```

ℹ️ Designbeslut: Edge Computing & Lagringsarkitektur (NVMe)
En honeypot-sensor som emulerar OT-miljöer och kör nätverksanalys genererar intensiva I/O-operationer. Ett traditionellt MicroSD-kort degraderas snabbt. Genom att utrusta Sentry-noden med en NVMe HAT och boota OS direkt från en M.2 SSD säkerställs industriell stabilitet och prestanda. Då Conpot är byggt för x86, hanterar Docker automatiskt emulering via qemu-user-static på vår ARM64-arkitektur.


---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


## 2️⃣ Docker & Infrastruktur

Noden använder Docker Compose för att snabbt och konsekvent rulla ut både honeypot (Conpot) och övervakningsstacken.

Steg:
Installera Docker Engine (via officiellt skript):

```bash

curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh
````

Ge din användare behörighet att köra Docker (för att slippa skriva sudo framför varje docker-kommando i framtiden):

```bash
sudo usermod -aG docker $USER

```
(Obs: Kräver ut- och inloggning för att gälla)


---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


## 3️⃣ Projektstruktur & Volymer

För att säkerställa att loggar och processdata från Sentry-noden inte försvinner om en container startas om, sätter vi upp en strikt katalogstruktur på NVMe-enheten (Bind Mounts).

Steg:
Skapa projektkatalog och underkataloger för persistens:

```bash
mkdir -p ~/nordhamn-ot/{conpot_logs,suricata_logs,loki_data,grafana_data}
cd ~/nordhamn-ot

```

ℹ️ Designbeslut: Data Persistence (Bind Mounts vs Volumes)
Istället för att låta Docker hantera anonyma volymer, används uttryckliga 'Bind Mounts' mappar på host-systemet. Detta gör det mycket enklare för en administratör att direkt komma åt, backa upp eller rensa specifika loggfiler (t.ex. PCAP-filer från Suricata) utan att behöva interagera med Dockers interna filsystem.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 4️⃣ Konfiguration av Custom XML-mall (Bypass av VFS)


Conpot v0.6.0 har strikta, odokumenterade XSD-scheman. Vi skapar en skräddarsydd HTTP-konfiguration som maskerar servern som Apache och dirigerar trafik till vår lokala HTML-fil, utan att krascha det virtuella filsystemet.



```bash
cat << 'EOF' > nordhamn/http/http.xml
<http enabled="True" host="0.0.0.0" port="8080">
    <global>
        <config>
            <entity name="server">Apache/2.4.41 (Ubuntu)</entity>
        </config>
        <headers>
            <entity name="Cache-Control">no-store, no-cache, must-revalidate</entity>
        </headers>
    </global>
    <htdocs>
        <node name="/"/>
    </htdocs>
    <statuscodes>
        <status name="404">
            <tarpit>0</tarpit>
            <entity name="body">404 Not Found</entity>
        </status>
    </statuscodes>
</http>
EOF

```

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 5️⃣ Web Portal Injection (High Interaction)



För att lura angripare injiceras en hårdkodad och trovärdig inloggningssida för "Nordhamn Kaj 8314". HTML-koden måste vara helt ren från otillåtna ASCII-tecken för att inte krascha simulatorns inbyggda Jinja2-motor.



```bash

cat << 'EOF' > nordhamn/http/htdocs/index.html

<!DOCTYPE html>

<html>

<head>

<title>Nordhamn Oil Terminal - PS-01</title>

<style>

body { font-family: Arial, sans-serif; background-color: #e0e0e0; text-align: center; margin-top: 100px; }

.login-box { background: white; width: 350px; margin: auto; padding: 30px; border: 2px solid #333; box-shadow: 5px 5px 15px #888; }

h2 { color: #003366; }

input[type="text"], input[type="password"] { width: 90%; padding: 10px; margin: 10px 0; border: 1px solid #ccc; }

input[type="submit"] { width: 95%; padding: 10px; background: #003366; color: white; border: none; font-weight: bold; cursor: pointer; }

input[type="submit"]:hover { background: #002244; }

.warning { font-size: 12px; color: red; margin-top: 20px; font-weight: bold; }

</style>

</head>

<body>

<div class="login-box">

<h2>Nordhamn Kaj 8314</h2>

<p><strong>Pump Station 01 (PS-01)</strong><br>Siemens S7 Web Interface</p>

<form action="/login_failed" method="POST">

<input type="text" name="username" placeholder="Operator ID" required>

<input type="password" name="password" placeholder="PIN Code" required>

<input type="submit" value="AUTHENTICATE">

</form>

<p class="warning">&#9888;&#65039; RESTRICTED OT NETWORK.<br>UNAUTHORIZED ACCESS IS STRICTLY PROHIBITED AND MONITORED.</p>

</div>

</body>

</html>

EOF

```



För att containerns begränsade användare ska kunna läsa filen sätter vi rättigheterna:



```bash

chmod -R 777 nordhamn/http/htdocs

```



ℹ️ Designbeslut: Web Portal Injection

Istället för att låta simulatorn spotta ut tomma sidor eller standardiserade felkoder, möts angriparen av en autentisk Siemens-inloggningssida. Detta ökar markant chansen att samla in värdefulla inloggningsförsök (brute-force data) då angriparen tror sig ha hittat ett kritiskt styrsystem.



---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------



## 6️⃣ Orchestration med Docker Compose

Slutligen knyts hela miljön ihop. Vi mappar in vår skräddarsydda nordhamn-katalog direkt in i containern som en template.


```bash

cat << 'EOF' > docker-compose.yml
version: '3.8'
services:
  conpot:
    image: honeynet/conpot:0.6.0
    container_name: nordhamn_conpot
    platform: linux/amd64
    restart: unless-stopped
    ports:
      - "80:8080"   # HTTP
      - "102:102"   # S7Comm
      - "502:502"   # Modbus
      - "161:161/udp" # SNMP
    volumes:
      - ./conpot_logs:/var/log/conpot
      - ./nordhamn:/nordhamn
    command: ["--template", "/nordhamn", "--logfile", "/var/log/conpot/conpot.log"]
EOF

```

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


##  Starta och Verifiera

Starta hela OT-nätverket i bakgrunden:


```bash
sudo docker compose up -d
```

Verifiera att webbservern och fällan fungerar genom att begära sidan lokalt:


```bash
curl -L http://localhost
```


---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 🚀 Driftsättning med Docker

För att köra Nordhamn OT-fällan på ett stabilt sätt (utan XML-valideringsfel från Conpots inbyggda motor) använder vi en skräddarsydd Docker Compose-konfiguration. Vi utnyttjar Docker Bind Mounts för att smyga in vår egen databashjärna (`template.xml`) och vårt grafiska gränssnitt (`index.html`) direkt över Conpots standardmall.

Skapa en fil med namnet `docker-compose.yml` i huvudmappen och klistra in följande konfiguration:

```bash
```yaml
services:
  conpot:
    image: ghcr.io/telekom-security/conpot:24.04.1
    container_name: nordhamn_conpot
    restart: unless-stopped
    environment:
      - CONPOT_TMP=/tmp
      - CONPOT_JSON_LOG=/var/log/conpot/conpot.json
    ports:
      - '80:80'       # HTTP (Webbgränssnitt / HMI)
      - '102:102'     # S7Comm (Siemens PLC)
      - '502:502'     # Modbus (Pumpstyrning)
      - '161:161/udp' # SNMP (Övervakning)
    volumes:
      - ./conpot_logs:/var/log/conpot
      - ./nordhamn/template.xml:/usr/lib/python3.11/site-packages/conpot/templates/default/template.xml
      - ./nordhamn/http/htdocs/index.html:/usr/lib/python3.11/site-packages/conpot/templates/default/http/htdocs/index.html
    command: conpot --template default --config /etc/conpot/conpot.cfg --logfile /var/log/conpot/conpot.log --temp_dir /tmp
```

När filerna är på plats, starta fällan i bakgrunden med kommandot:

```bash
sudo docker compose up -d
```

För att öka trovärdigheten (och fånga upp HTTP-baserade skanningar) serverar fällan ett inloggningsgränssnitt på port 80. Sidan utger sig för att vara en fjärråtkomst till Nordhamns centrala pumpstyrning och fungerar som ett utmärkt lockbete för angripare.

(Trafik och inloggningsförsök på denna sida loggas omedelbart av Conpot).

