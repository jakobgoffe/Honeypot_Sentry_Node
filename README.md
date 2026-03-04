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
Detta projekt fokuserar på att emulera de industriella styrsystemen (OT) vid en simulerad hamn "Nordhamn" och dess Oljeterminal, Kaj 8314. Syftet är att skapa en höginteraktiv honeypot som simulerar en Siemens S7-PLC för att studera exponering av kritiska protokoll och nätverksinteraktioner i en kontrollerad miljö.

## 🧠 Metodik & AI som "Force Multiplier"
Min kärnkompetens och mitt primära fokus i detta projekt ligger på den teoretiska förståelsen för OT-säkerhet – arkitektur, nätverkssegmentering, hotmodellering och driftsäkerhet.

För att översätta denna teoretiska "VAD"och "VARFÖR"-kunskap till praktiskt "HUR" (avancerad Linux-härdning, felsökning av källkod och Docker-orkestrering) har jag aktivt använt LLM-verktyg som en teknisk sparringpartner. Detta projekt demonstrerar därmed inte bara implementering av OT-säkerhetsprinciper i praktiken, utan bevisar också förmågan att agilt använda modern AI för att snabbt och säkert realisera komplexa arkitekturer.

## 🛠️ Teknisk Stack
- **Simulator:** Conpot v0.6.0 (MushMush Foundation)
- **Monitoring & IDS:** Suricata (Intrusion Detection), Promtail & Loki
- **Visualisering:** Grafana
- **Infrastruktur:** Docker & Docker Compose
- **Hårdvara:** Raspberry Pi (Sentry-nod)
- **Protokoll:** S7Comm, Modbus, SNMP, BACnet, HTTP

## 🚀 Implementerade Funktioner
- [x] **Containeriserad miljö:** Fullt konfigurerad Docker-stack för snabb driftsättning.
- [x] **Custom PLC Template:** Skräddarsydd XML-mall för Nordhamns specifika identitet (Kaj 8314).
- [x] **S7Comm Emulering:** Port 102 exponerad med Siemens-specifika svarstider.
- [x] **VFS Optimering:** Implementering av minnesbaserat filsystem (`mem://`) för ökad stabilitet i containermiljön.

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

ℹ️ Designbeslut: Lagringsarkitektur & Prestanda (NVMe)

En honeypot-sensor som emulerar OT-miljöer (Conpot) och samtidigt kör nätverksanalys (Suricata) genererar intensiva I/O-operationer (läs/skriv).

Slitage: Ett traditionellt MicroSD-kort degraderas snabbt i denna typ av miljö och riskerar att korrumpera filsystemet.

Stabilitet: Genom att utrusta Sentry-noden med en NVMe HAT och boota OS direkt från en M.2 SSD säkerställs industriell stabilitet, hög prestanda för Docker-containrar och tillräcklig kapacitet för centraliserad loggning.


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

## 4️⃣ Konfiguration & Honeypot-mallen

För att Conpot ska agera trovärdigt måste vi injicera en anpassad identitet. Standardmallarna i Conpot 0.6.0 lider dessutom av buggar relaterade till filsystemet som vi måste by-passa.

Steg1 :
Skapa konfigurationsfiler och injicera XML-mallen som definierar Nordhamns PLC (Siemens S7-300).

```bash
touch conpot.cfg nordhamn_working.xml nordhamn_index.html
```

ℹ️ Designbeslut: VFS Optimering (mem://)
En känd instabilitet i simulatorn är att moduler som FTP och HTTP försöker skriva till ett låst virtuellt filsystem (VFS), vilket leder till "ResourceReadOnly"-krascher. Genom att explicit styra om data_fs_url till mem:// i XML-mallen, tvingar vi honeypoten att hantera alla filoperationer temporärt i RAM-minnet, vilket eliminerar krascherna helt.

Steg 2: 
Injicera en anpassad HTML-portal för webbgränssnittet (Port 80) för att maskera simulatorns bakomliggande teknik:

```bash
# Skapar en industriell inloggningssida för Nordhamn Kaj 8314
cat << 'EOF' > nordhamn_index.html
EOF
```

ℹ️ Designbeslut: Web Portal Injection
Då Conpot v0.6.0 visade sig ha kompatibilitetsproblem med sitt inbyggda Jinja2-mallsystem, implementerades en strategisk override. Genom att volymmontera (bind mount) en statisk HTML-fil över containerns standard-index skapas en högre grad av realism (High Interaction Honeypot). Angriparen möts av en autentisk Siemens-inloggningssida för "Nordhamn" istället för ett tomt svar, vilket ökar chansen att samla in värdefulla inloggningsförsök (brute-force data).
