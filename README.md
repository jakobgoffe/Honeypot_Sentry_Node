# Honeypot_Sentry_Node
Detta projekt fokuserar på att emulera de industriella styrsystemen (OT) vid en simulerad hamn "Nordhamn" och dess Oljeterminal, Kaj 8314**. Syftet är att skapa en höginteraktiv honeypot som simulerar en Siemens S7-PLC för att studera exponering av kritiska protokoll och nätverksinteraktioner i en kontrollerad miljö.
# ⚓ Nordhamn Oil Transfer (OT) Simulator

<div align="center">
  <img src="https://img.shields.io/badge/Status-Development-orange?style=for-the-badge" alt="Status">
  <img src="https://img.shields.io/badge/Platform-Raspberry%20Pi-C51A4A?style=for-the-badge&logo=raspberry-pi" alt="Raspberry Pi">
  <img src="https://img.shields.io/badge/Docker-Enabled-2496ED?style=for-the-badge&logo=docker" alt="Docker">
</div>

---

## 📖 Om Projektet
Detta projekt fokuserar på att emulera de industriella styrsystemen (OT) vid **Nordhamn Oljeterminal, Kaj 8314**. Syftet är att skapa en höginteraktiv honeypot som simulerar en Siemens S7-PLC för att studera exponering av kritiska protokoll och nätverksinteraktioner i en kontrollerad miljö.

## 🛠️ Teknisk Stack
- **Simulator:** Conpot v0.6.0 (MushMush Foundation)
- **Infrastruktur:** Docker & Docker Compose
- **Hårdvara:** Raspberry Pi (Sentry-nod)
- **Protokoll:** S7Comm, Modbus, SNMP, BACnet, HTTP

## 🚀 Implementerade Funktioner
- [x] **Containeriserad miljö:** Fullt konfigurerad Docker-stack för snabb driftsättning.
- [x] **Custom PLC Template:** Skräddarsydd XML-mall för Nordhamns specifika identitet (Kaj 8314).
- [x] **S7Comm Emulering:** Port 102 exponerad med Siemens-specifika svarstider.
- [x] **VFS Optimering:** Implementering av minnesbaserat filsystem (`mem://`) för ökad stabilitet i containermiljön.

## ⚠️ Utmaningar & Lärdomar (LIA-fokus)
Under utvecklingsfasen har stor vikt lagts vid att lösa kompatibilitetsproblem i **Conpot 0.6.0**. Arbetet har gett djupgående kunskaper i:
- **Konfigurationsvalidering:** Felsökning av `configparser` och källkod för att möta strikta krav på sektionsindelning i .cfg-filer.
- **Protocol Handshaking:** Analys av varför vissa Nmap-skript (t.ex. `s7-info`) nekas vid emulering och hur man anpassar svar för att öka trovärdigheten.
- **Data Bus Injection:** Pågående arbete med att synkronisera simulatorns interna variabler med det externa webbgränssnittet.

## 🔧 Installation & Användning
1. Klona repot till din Raspberry Pi:
   ```bash
   git clone [https://github.com/jakobgoffe/nordhamn-ot.git](https://github.com/jakobgoffe/nordhamn-ot.git)
