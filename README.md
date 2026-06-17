# 📡 Supervision réseau avec SNMP & LibreNMS

> TP de supervision réseau basé sur le protocole SNMP — Ubuntu 22.04 LTS  
> Rapport TP — ESTM Dakar | L3 Réseaux & Télécommunications | 2025-2026

---

## 📌 Description

Déploiement complet d'un agent SNMP sur Ubuntu, exploration de la MIB, sécurisation avec SNMPv3, automatisation de la collecte de métriques via un script cron, et intégration dans **LibreNMS** via Docker Compose.

---

## ✅ Réalisations

### Partie 1 — Installation & configuration agent SNMP
- Installation `snmpd` + outils SNMP sur Ubuntu
- Configuration community string `estm` en lecture seule (`rocommunity`)
- Chargement des MIBs (`snmp-mibs-downloader`)
- Test : `snmpget -v2c -c estm localhost sysDescr.0`

### Partie 2 — Requêtes & exploration MIB
- Informations système : `snmpget` (sysName, sysDescr, sysContact)
- Liste des interfaces : `snmpwalk -v2c -c estm localhost ifDescr`
- Compteurs trafic : `ifInOctets` / `ifOutOctets`
- Modification `sysContact` via `snmpset` avec accès RW

### Partie 3 — SNMPv3 (sécurisé)
- Création utilisateur SNMPv3 avec **authentification SHA** + **chiffrement AES**
- Test accès sécurisé : `snmpget -v3 -l authPriv -u maty -a SHA -x AES`
- Validation du refus d'accès non autorisé (timeout sur mauvaise community)

### Partie 4 — Script de supervision automatisé
```bash
#!/bin/bash
# snmp_monitor.sh — Collecte CPU, mémoire, trafic réseau toutes les 5 min
HOST="localhost"
COMMUNITY="estm"
LOGFILE="/var/log/snmp_monitor.log"

SYSNAME=$(snmpget -v2c -c $COMMUNITY $HOST sysName.0 -Oqv)
UPTIME=$(snmpget -v2c -c $COMMUNITY $HOST sysUpTime.0 -Oqv)
INOCTS=$(snmpget -v2c -c $COMMUNITY $HOST ifInOctets.2 -Oqv)
OUTOCTS=$(snmpget -v2c -c $COMMUNITY $HOST ifOutOctets.2 -Oqv)

echo "[$TIMESTAMP] Hote: $SYSNAME | Uptime: $UPTIME | In: $INOCTS | Out: $OUTOCTS" >> $LOGFILE
```
- Planification crontab : `*/5 * * * * /usr/local/bin/snmp_monitor.sh`
- Logs vérifiés via `tail -20 /var/log/syslog | grep -i snmp`

### Partie 5 — LibreNMS via Docker Compose
- Déploiement LibreNMS (MariaDB, Redis, Dispatcher, Syslogng, SNMPtrapd)
- Accès interface web : `http://192.168.69.131:8000`
- Ajout équipement supervisé (community `estm`, SNMPv2c)
- Graphiques trafic réseau générés automatiquement (interface ens33)

---

## ⚙️ Stack technique

| Outil | Rôle |
|-------|------|
| `snmpd` (Net-SNMP) | Agent SNMP sur Ubuntu |
| SNMPv2c / SNMPv3 | Protocoles de supervision |
| LibreNMS | Dashboard de supervision réseau |
| Docker Compose | Déploiement LibreNMS conteneurisé |
| Crontab | Automatisation collecte métriques |

---

## 🔑 Commandes clés

```bash
# Installer l'agent SNMP
sudo apt install -y snmp snmpd snmp-mibs-downloader

# Requête de base (v2c)
snmpget -v2c -c estm localhost sysDescr.0

# Walk des interfaces
snmpwalk -v2c -c estm localhost ifDescr

# Requête SNMPv3 sécurisée
snmpget -v3 -l authPriv -u maty -a SHA -A "MotDePasseAuth" \
        -x AES -X "MotDePassePriv" localhost sysName.0

# Déployer LibreNMS
git clone https://github.com/librenms/docker.git librenms-docker
cd librenms-docker/examples/compose
sudo docker compose up -d
```

---

## 👩‍💻 Auteure

**Ndèye Maty Gueye** — L3 Réseaux & Télécommunications, ESTM Dakar  
[![Email](https://img.shields.io/badge/Email-ndeyematygueye5%40gmail.com-blue?style=flat&logo=gmail)](mailto:ndeyematygueye5@gmail.com)

---

> TP réalisé sous la direction du **Dr. Kéba GUEYE** — ESTM Dakar 2025-2026
