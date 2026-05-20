# 🔬 Collection de LABs Wireshark pour SOC Analyste

## 📖 À Propos

Cette collection contient **8 fichiers PCAP réalistes** simulant les incidents de sécurité les plus courants que vous rencontrerez en tant qu'analyste SOC. Chaque LAB est accompagné d'instructions détaillées d'analyse.

## 🎯 LABs Disponibles

| # | Nom | Fichier | Scénario | Difficulté |
|---|-----|---------|----------|-----------|
| 1 | **Port Scan** | `LAB1_Port_Scan.pcap` | Reconnaissance Nmap | ⭐ Débutant |
| 2 | **SSH Brute Force** | `LAB2_SSH_BruteForce.pcap` | Attaque par dictionnaire + Compromission | ⭐⭐ Intermédiaire |
| 3 | **DNS Tunneling** | `LAB3_DNS_Tunneling.pcap` | Exfiltration de données encodées | ⭐⭐⭐ Avancé |
| 4 | **C2 Beacon** | `LAB4_C2_Beacon.pcap` | Communication périodique avec C2 | ⭐⭐ Intermédiaire |
| 5 | **Lateral Movement** | `LAB5_Lateral_Movement.pcap` | Propagation via SMB/PsExec | ⭐⭐⭐ Avancé |
| 6 | **Malware Download** | `LAB6_Malware_Download.pcap` | Drive-by download + Callback | ⭐⭐ Intermédiaire |
| 7 | **SQL Injection** | `LAB7_SQL_Injection.pcap` | Exploitation web avec SQLMap | ⭐⭐ Intermédiaire |
| 8 | **ARP Spoofing** | `LAB8_ARP_Spoofing.pcap` | Man-in-the-Middle + Interception | ⭐⭐⭐ Avancé |

## 🚀 Démarrage Rapide

### 1. Télécharger Wireshark
```bash
# Ubuntu/Debian
sudo apt install wireshark

# macOS (avec Homebrew)
brew install --cask wireshark

# Windows
# Télécharger depuis : https://www.wireshark.org/download.html
```

### 2. Ouvrir un LAB
1. Lancer Wireshark
2. `File → Open`
3. Sélectionner un fichier `.pcap`
4. Commencer l'analyse avec le guide

### 3. Consulter le Guide
Ouvrez `GUIDE_ANALYSE_LABS.md` pour des instructions détaillées étape par étape.

## 📚 Structure des Fichiers

```
wireshark_labs/
├── README.md                      # Ce fichier
├── GUIDE_ANALYSE_LABS.md          # Guide complet d'analyse
├── LAB1_Port_Scan.pcap           # Scan de ports
├── LAB2_SSH_BruteForce.pcap      # Brute force SSH
├── LAB3_DNS_Tunneling.pcap       # Exfiltration DNS
├── LAB4_C2_Beacon.pcap           # Communication C2
├── LAB5_Lateral_Movement.pcap    # Mouvement latéral
├── LAB6_Malware_Download.pcap    # Téléchargement malware
├── LAB7_SQL_Injection.pcap       # Injection SQL
└── LAB8_ARP_Spoofing.pcap        # ARP Spoofing
```

## 🎓 Compétences Développées

Après avoir complété ces LABs, vous saurez :

✅ Détecter une reconnaissance réseau (scans de ports)  
✅ Identifier une attaque par force brute et confirmer une compromission  
✅ Reconnaître une exfiltration via DNS tunneling  
✅ Repérer des communications C2 périodiques  
✅ Tracer un mouvement latéral dans le réseau  
✅ Analyser un téléchargement de malware et extraire les artefacts  
✅ Identifier des injections SQL et les données exposées  
✅ Détecter un ARP spoofing et un Man-in-the-Middle  

## 🔍 Filtres Wireshark Essentiels

### Détection d'Anomalies
```
tcp.flags.syn == 1                    # Scans de ports
tcp.port == 22                        # Trafic SSH
dns && (frame.len > 100)              # DNS tunneling
http.request.uri contains "SELECT"    # SQL Injection
arp.duplicate-address-detected        # ARP Spoofing
```

### Analyse de Trafic
```
ip.addr == 192.168.1.50              # Trafic d'une IP
tcp.port == 445                       # SMB (lateral movement)
http.request.method == "GET"          # Requêtes HTTP GET
dns.qry.name contains "evil"          # Domaines suspects
```

## 🏆 Challenges

### Challenge 1 : Speed Run
Analysez **LAB1** en moins de 5 minutes et identifiez :
- IP de l'attaquant
- Type de scan
- Ports ouverts découverts

### Challenge 2 : Compromission Cachée
Dans **LAB2**, trouvez la connexion SSH réussie parmi les 500 tentatives.
Indice : Cherchez la durée anormale.

### Challenge 3 : Décodage Complet
**LAB3** : Décodez TOUT le message exfiltré via DNS.
Combien de caractères contient-il ?

### Challenge 4 : Timeline Forensics
**LAB5** : Créez une timeline précise montrant l'ordre de compromission des 4 machines.

### Challenge 5 : Credential Recovery
**LAB8** : Interceptez les credentials HTTP en clair et craquez le hash MD5 de **LAB7**.

## 📊 Métriques de Progression

| LAB | Temps Moyen | IOCs à Identifier | Difficulté Technique |
|-----|-------------|-------------------|----------------------|
| LAB 1 | 10-15 min | 4 | Débutant |
| LAB 2 | 15-20 min | 5 | Intermédiaire |
| LAB 3 | 20-30 min | 6 | Avancé |
| LAB 4 | 15-20 min | 4 | Intermédiaire |
| LAB 5 | 20-25 min | 7 | Avancé |
| LAB 6 | 15-20 min | 6 | Intermédiaire |
| LAB 7 | 15-20 min | 5 | Intermédiaire |
| LAB 8 | 20-30 min | 6 | Avancé |

## 🛠️ Outils Complémentaires

Pour aller plus loin dans l'analyse :

- **tshark** : Version CLI de Wireshark (automatisation)
- **NetworkMiner** : Extraction automatique d'artefacts
- **Zeek** : Analyse réseau avancée avec logs
- **Suricata** : IDS/IPS avec règles personnalisées
- **CyberChef** : Décodage/encodage (base64, hex, etc.)

## 📖 Ressources d'Apprentissage

### Documentation Officielle
- [Wireshark User Guide](https://www.wireshark.org/docs/)
- [Display Filter Reference](https://www.wireshark.org/docs/dfref/)
- [Sample Captures](https://wiki.wireshark.org/SampleCaptures)

### Sites de Pratique
- **Malware-Traffic-Analysis.net** : PCAPs réels d'infections
- **CyberDefenders.org** : Challenges Blue Team
- **TryHackMe** : Rooms Wireshark interactives
- **HackTheBox** : Challenges Sherlock (DFIR)

### Chaînes YouTube
- **Chris Greer** : Tutorials Wireshark
- **David Bombal** : Networking & Security
- **13Cubed** : DFIR & Network Forensics

## 🎯 Parcours d'Apprentissage Recommandé

### Semaine 1 : Fondamentaux
- LAB 1 : Port Scan
- LAB 6 : Malware Download
- LAB 4 : C2 Beacon

### Semaine 2 : Attaques Applicatives
- LAB 2 : SSH Brute Force
- LAB 7 : SQL Injection

### Semaine 3 : Techniques Avancées
- LAB 3 : DNS Tunneling
- LAB 5 : Lateral Movement
- LAB 8 : ARP Spoofing

### Semaine 4 : Analyse Complète
- Exercice final : Analyser un incident multi-étapes
- Créer vos propres règles de détection
- Automatiser l'analyse avec tshark

## ⚠️ Avertissement

Ces fichiers PCAP sont des **simulations pédagogiques** créées avec Scapy :
- ✅ Utilisables librement dans un cadre éducatif
- ✅ Ne contiennent AUCUNE donnée réelle ou sensible
- ✅ Conçus pour l'apprentissage de l'analyse réseau
- ❌ Ne reflètent pas toute la complexité du trafic réel
- ❌ Volumes de données réduits (pour faciliter l'apprentissage)

## 🤝 Contribution

Ces LABs ont été créés dans un but pédagogique. Si vous avez des suggestions d'amélioration ou souhaitez contribuer de nouveaux scénarios, n'hésitez pas !

## 📞 Support

Pour toute question sur l'utilisation de ces LABs :
- Consultez d'abord le `GUIDE_ANALYSE_LABS.md`
- Vérifiez la documentation officielle Wireshark
- Rejoignez les communautés SOC/Blue Team (Reddit, Discord)

## 📜 Licence

Ces fichiers PCAP sont fournis à des fins éducatives uniquement.
Libre d'utilisation pour la formation et l'apprentissage.

---

**Bon apprentissage et excellente chasse aux incidents ! 🔍🛡️**

*"The network never lies. It just needs someone to listen."*

## 🏅 Certification

Après avoir maîtrisé ces LABs, vous serez prêt pour :
- **Wireshark Certified Network Analyst (WCNA)**
- **CompTIA CySA+** (Cybersecurity Analyst)
- **GIAC Certified Intrusion Analyst (GCIA)**
- Postes de **SOC Analyst Level 1/2**

---

Dernière mise à jour : Mai 2024
Version : 1.0

---

## 🛡️ NOUVEAU : Règles de Détection Wazuh

### 📄 WAZUH_DETECTION_RULES.md

En complément des LABs Wireshark, nous avons ajouté un fichier complet avec **52 règles de détection Wazuh** prêtes à déployer en production !

#### Ce que vous obtenez :

✅ **Règles XML complètes** pour chaque type d'attaque  
✅ **Décodeurs personnalisés** pour parser les logs  
✅ **Scripts Python** d'analyse avancée (entropie DNS, périodicité C2)  
✅ **Active Response** pour blocage automatique des attaquants  
✅ **Intégration MITRE ATT&CK** pour chaque règle  
✅ **Configuration dashboards** pour monitoring  
✅ **Gestion des faux positifs** avec whitelisting  

#### Règles par LAB :

| LAB | Règles | Description |
|-----|--------|-------------|
| 1 | 5 règles | Détection scan de ports (SYN, intensif, ports critiques) |
| 2 | 6 règles | Détection brute force SSH + compromission |
| 3 | 6 règles | Détection DNS tunneling (entropie, volume, encodage) |
| 4 | 6 règles | Détection C2 beacon (périodicité, certificats suspects) |
| 5 | 7 règles | Détection lateral movement (SMB, PsExec, WMI) |
| 6 | 7 règles | Détection malware download + intégration VirusTotal |
| 7 | 9 règles | Détection SQL injection (tous types, outils, erreurs) |
| 8 | 6 règles | Détection ARP spoofing (MITM, duplicate ARP) |

#### Exemple de Règle :

```xml
<!-- Détection de Brute Force SSH -->
<rule id="100011" level="10" frequency="10" timeframe="120">
  <if_matched_sid>100010</if_matched_sid>
  <same_source_ip />
  <description>Brute force SSH détecté - 10+ échecs depuis $(srcip)</description>
  <mitre>
    <id>T1110.001</id>
  </mitre>
</rule>

<!-- Blocage automatique -->
<active-response>
  <command>firewall-drop</command>
  <rules_id>100011</rules_id>
  <timeout>3600</timeout>
</active-response>
```

#### Déploiement :

```bash
# Copier les règles dans Wazuh
sudo nano /var/ossec/etc/rules/local_rules.xml

# Redémarrer
sudo systemctl restart wazuh-manager

# Tester
/var/ossec/bin/wazuh-logtest
```

### 🎯 Vision Complète SOC

En combinant **Wireshark (analyse) + Wazuh (détection)**, vous maîtrisez :

1. **Post-incident** : Analyser les PCAP avec Wireshark
2. **Temps réel** : Détecter avec les règles Wazuh
3. **Réponse automatique** : Bloquer les attaquants
4. **Conformité** : Mapping MITRE ATT&CK + PCI-DSS/GDPR

### 📊 Intégration MITRE ATT&CK

Toutes les règles sont mappées aux techniques MITRE :

- **T1046** - Network Service Discovery (Port Scan)
- **T1110.001** - Password Guessing (Brute Force)
- **T1048.003** - Exfiltration via DNS
- **T1071.001** - Web Protocols (C2)
- **T1021.002** - SMB/Windows Admin Shares
- **T1204.002** - Malicious File
- **T1190** - Exploit Public-Facing Application
- **T1557.002** - ARP Cache Poisoning

**→ Coverage de 8 techniques MITRE essentielles !**

---

