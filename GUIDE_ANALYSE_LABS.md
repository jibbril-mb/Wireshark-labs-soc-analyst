# 🔬 Guide Complet d'Analyse des LABs Wireshark

## 📥 Téléchargement des Fichiers PCAP

Tous les fichiers PCAP ont été générés et sont prêts à être analysés avec Wireshark.

### Liste des Fichiers

| LAB | Fichier | Taille | Scénario |
|-----|---------|--------|----------|
| **LAB 1** | `LAB1_Port_Scan.pcap` | ~2000 paquets | Scan de ports Nmap |
| **LAB 2** | `LAB2_SSH_BruteForce.pcap` | ~5100 paquets | Attaque par force brute SSH |
| **LAB 3** | `LAB3_DNS_Tunneling.pcap` | ~50 paquets | Exfiltration via DNS |
| **LAB 4** | `LAB4_C2_Beacon.pcap` | ~300 paquets | Communication C2 périodique |
| **LAB 5** | `LAB5_Lateral_Movement.pcap` | ~76 paquets | Mouvement latéral SMB |
| **LAB 6** | `LAB6_Malware_Download.pcap` | ~50 paquets | Téléchargement de malware |
| **LAB 7** | `LAB7_SQL_Injection.pcap` | ~58 paquets | Attaque SQL Injection |
| **LAB 8** | `LAB8_ARP_Spoofing.pcap` | ~40 paquets | ARP Spoofing (MITM) |

---

## 🎯 LAB 1 : Détection de Scan de Ports

### 📋 Objectifs
- Identifier l'IP de l'attaquant
- Déterminer le type de scan utilisé
- Découvrir les ports ouverts sur la cible
- Estimer la durée du scan

### 🔍 Instructions d'Analyse

#### Étape 1 : Ouvrir le fichier
```
Fichier → Ouvrir → LAB1_Port_Scan.pcap
```

#### Étape 2 : Filtrer le trafic suspect
```
Filtre Wireshark : tcp.flags.syn == 1
```

**🤔 Que recherchez-vous ?**
- Beaucoup de paquets SYN depuis une même IP
- Destination vers de multiples ports
- Timing très rapide entre les paquets

#### Étape 3 : Analyser les conversations
```
Menu : Statistics → Conversations → TCP
Trier par : "Packets" (décroissant)
```

**📊 Ce que vous devriez voir :**
- Une IP source domine (192.168.1.50)
- Multiples ports de destination (1-1000)
- Conversations courtes (2-4 paquets)

#### Étape 4 : Identifier les ports ouverts
```
Filtre : tcp.flags.syn == 1 && tcp.flags.ack == 1 && ip.src == 192.168.1.100
```

**💡 Explication :**
- `SYN-ACK` = le serveur accepte la connexion
- Ce sont les ports OUVERTS découverts par l'attaquant

**✅ Réponses attendues :**
- Ports ouverts : 22 (SSH), 80 (HTTP), 443 (HTTPS), 3306 (MySQL)
- Port 3306 ne devrait PAS être exposé → **Vulnérabilité critique**

#### Étape 5 : Timeline de l'attaque
```
Menu : Statistics → IO Graph
Filter : tcp.flags.syn == 1 && ip.src == 192.168.1.50
Y-Axis : COUNT
Interval : 1 second
```

**📈 Analyse :**
- Pic soudain d'activité = début du scan
- Vitesse : ~1000 paquets en 1 seconde
- Signature typique d'un outil automatisé (Nmap)

---

## 🎯 LAB 2 : Brute Force SSH

### 📋 Objectifs
- Compter le nombre de tentatives de connexion
- Identifier si une connexion a réussi
- Calculer la vitesse d'attaque

### 🔍 Instructions d'Analyse

#### Étape 1 : Filtrer le trafic SSH
```
Filtre : tcp.port == 22 && ip.dst == 192.168.1.100
```

#### Étape 2 : Compter les tentatives
```
Menu : Statistics → Conversations → TCP
```

**🔢 Comptage :**
- Nombre de lignes avec port de destination 22
- Chaque ligne = 1 tentative de connexion

**✅ Réponse attendue : ~500 tentatives**

#### Étape 3 : 🚨 DÉTECTER LA COMPROMISSION
```
Menu : Statistics → Conversations → TCP
Trier par : "Duration" (décroissant)
```

**💡 Logique de détection :**
| Durée | Interprétation |
|-------|----------------|
| < 5 secondes | Échec d'authentification |
| > 30 secondes | ⚠️ CONNEXION RÉUSSIE |

**🔍 Dans ce LAB :**
- Cherchez une connexion avec durée > 800 secondes
- Port source : 52458
- **Cette connexion = compromission confirmée**

#### Étape 4 : Analyser la connexion compromise
```
Filtre : tcp.stream eq [numéro du stream long]
Clic droit → Follow → TCP Stream
```

**📋 Ce que vous verrez :**
- Échange SSH initial
- Données chiffrées (session interactive)
- Volume important de paquets échangés
- L'attaquant a obtenu un shell !

#### Étape 5 : Timeline de l'attaque
```
Statistics → IO Graph
Filter : tcp.port == 22 && tcp.flags.syn == 1
Interval : 10 seconds
```

**📊 Pattern attendu :**
- ~50 tentatives/minute
- Rythme régulier (automatisé)
- Une pause peut indiquer le moment du succès

---

## 🎯 LAB 3 : DNS Tunneling

### 📋 Objectifs
- Détecter l'exfiltration de données via DNS
- Identifier le domaine malveillant
- Décoder le message exfiltré

### 🔍 Instructions d'Analyse

#### Étape 1 : Vue d'ensemble DNS
```
Filtre : dns
```

**👀 Observation initiale :**
- Beaucoup de requêtes DNS similaires
- Domaine répétitif : `strange-domain.xyz`

#### Étape 2 : Statistiques DNS
```
Menu : Statistics → DNS
Onglet : Queries
```

**🚨 Anomalie détectée :**
- `strange-domain.xyz` devrait dominer largement
- Comparer avec domaines légitimes (google, office365)

#### Étape 3 : Filtrer les requêtes suspectes
```
Filtre : dns.qry.name contains "strange-domain"
```

**📋 Ce que vous devriez voir :**
```
Q2xldmVyVGhpbmcuYmFzZTY0.strange-domain.xyz
ZW5jb2RpbmdJc0hlcmU.strange-domain.xyz
...
```

#### Étape 4 : Détecter les sous-domaines longs
```
Filtre : dns && (frame.len > 100)
```

**💡 Pourquoi c'est suspect ?**
- Requête DNS normale : 60-80 octets
- Avec données encodées : > 100 octets

#### Étape 5 : 🔓 Décoder le message secret

**Méthode manuelle :**
1. Sélectionner une requête DNS
2. Développer : `Domain Name System → Queries`
3. Copier le sous-domaine avant le premier `.`
4. Décoder avec un outil base64 :

```bash
# Linux / macOS
echo "Q09ORklERU5USUFM" | base64 -d

# Résultat : CONFIDENTIAL
```

**Méthode automatisée (si tshark disponible) :**
```bash
tshark -r LAB3_DNS_Tunneling.pcap -Y "dns.qry.name contains strange-domain" \
       -T fields -e dns.qry.name | \
       cut -d'.' -f1 | \
       while read line; do echo "$line" | base64 -d 2>/dev/null; done
```

**✅ Message complet attendu :**
```
CONFIDENTIAL: warehouse-financial-report-Q1-2024.pdf 
Budget: $12.5M, Salaries: $8.2M, Profit: $4.3M
```

#### Étape 6 : Identifier le serveur C2
```
Filtre : dns && dns.flags.response == 1
```

**🔍 Recherchez :**
- Quelle IP répond aux requêtes `strange-domain.xyz`
- **Réponse attendue : 203.0.113.100**

---

## 🎯 LAB 4 : C2 Beacon Communication

### 📋 Objectifs
- Identifier la périodicité des connexions
- Détecter le serveur C2
- Différencier du trafic HTTPS légitime

### 🔍 Instructions d'Analyse

#### Étape 1 : Filtrer HTTPS
```
Filtre : tcp.port == 443
```

#### Étape 2 : Analyser les conversations
```
Statistics → Conversations → TCP
Trier par : "Address B" (regrouper par destination)
```

**🔍 Recherchez :**
- Une IP de destination qui apparaît régulièrement
- Multiples connexions courtes

#### Étape 3 : Visualiser la périodicité
```
Statistics → IO Graph

Configuration :
- Filter : ip.dst == 203.0.113.200 && tcp.port == 443
- Y-Axis : COUNT(tcp.stream)
- Interval : 10 seconds
```

**📊 Pattern de beacon :**
```
Connexions
    |
  1 |  █     █     █     █     █
    |
  0 |__________________________
     0s    60s   120s  180s  240s
```

**💡 Intervalle régulier = 60 secondes**

#### Étape 4 : Comparer avec trafic légitime

**Trafic légitime (Google, Microsoft) :**
- Timing irrégulier
- Connexions plus longues
- Volume de données variable

**Trafic C2 :**
- ⏰ Timing RÉGULIER (60s dans ce LAB)
- Connexions courtes (<1 seconde)
- Volume de données constant

#### Étape 5 : Analyser un beacon
```
Sélectionner une connexion vers 203.0.113.200
Clic droit → Follow → TCP Stream
```

**🔍 Observation :**
- TLS Handshake
- Petite quantité de données chiffrées
- Fermeture immédiate

**✅ Conclusion : Machine compromise communiquant avec un C2**

---

## 🎯 LAB 5 : Lateral Movement via SMB

### 📋 Objectifs
- Identifier la machine compromise initiale
- Tracer la propagation vers d'autres machines
- Détecter l'utilisation de PsExec

### 🔍 Instructions d'Analyse

#### Étape 1 : Filtrer le trafic SMB
```
Filtre : tcp.port == 445
```

#### Étape 2 : Cartographier les connexions
```
Statistics → Conversations → TCP
```

**📊 Pattern suspect :**
- Une IP source (192.168.1.50) vers multiples destinations
- Destinations : 192.168.1.101, .102, .103, .104

**🚨 Anomalie :**
- Communication poste-à-poste (pas vers serveur de fichiers)
- C'est du mouvement latéral !

#### Étape 3 : Détecter PsExec
```
Filtre : smb2 || smb
```

**🔍 Dans la liste des paquets, cherchez :**
- `Create Request` avec des noms de fichiers
- Développer les détails du paquet

**Recherche manuelle :**
```
Menu : Edit → Find Packet
Find by : String
Search in : Packet Bytes
Search : PSEXESVC
```

**✅ Si trouvé : Preuve d'utilisation de PsExec**

#### Étape 4 : Identifier les partages administratifs
```
Filtre : smb2.filename contains "ADMIN$" || smb2.filename contains "C$"
```

**💡 Explication :**
- `ADMIN$` et `C$` = partages administratifs Windows
- Accès = privilèges administrateur
- Utilisation = mouvement latéral

#### Étape 5 : Timeline de propagation
```
Statistics → Conversations → TCP
Noter les timestamps de chaque nouvelle connexion
```

**📅 Timeline attendue :**
```
15:20:00 → 192.168.1.101 (1ère victime)
15:22:00 → 192.168.1.102 (2ème victime)
15:24:00 → 192.168.1.103 (3ème victime)
15:26:00 → 192.168.1.104 (4ème victime)
```

**⏱️ Intervalle : 2 minutes entre chaque infection**

---

## 🎯 LAB 6 : Téléchargement de Malware

### 📋 Objectifs
- Tracer la chaîne d'infection complète
- Extraire le fichier malveillant
- Identifier le callback C2 post-infection

### 🔍 Instructions d'Analyse

#### Étape 1 : Identifier la connexion initiale
```
Filtre : http
```

**🔍 Premier paquet HTTP :**
```
GET /blog/article.php?id=123 HTTP/1.1
Host: malicious-blog.com
```

**💡 C'est le site compromis visité par la victime**

#### Étape 2 : Détecter la redirection
```
Filtre : http.response.code == 302
```

**📋 Réponse HTTP :**
```
HTTP/1.1 302 Found
Location: http://203.0.113.150/downloads/update.exe
```

**🚨 Alerte : Redirection vers un fichier .exe**

#### Étape 3 : Analyser le téléchargement
```
Filtre : http.request.uri contains ".exe"
```

**🔍 Requête :**
```
GET /downloads/update.exe HTTP/1.1
```

**🦠 Nom du malware : `update.exe` (faux update Windows)**

#### Étape 4 : 📥 Extraire le fichier malveillant
```
Menu : File → Export Objects → HTTP
```

**Instructions :**
1. Dans la liste, chercher `update.exe`
2. Sélectionner → "Save As" → sauvegarder

**⚠️ ATTENTION : Fichier potentiellement dangereux !**
- Ne PAS exécuter
- Analyser dans une VM isolée ou avec VirusTotal

#### Étape 5 : Vérifier le header PE
```
Sélectionner le paquet avec Content-Type: application/x-msdownload
Clic droit → Follow → TCP Stream
```

**🔍 Recherchez au début du contenu :**
```
MZ.....
```

**💡 `MZ` = Magic Number des exécutables Windows (.exe)**
- Confirme que c'est bien un exécutable

#### Étape 6 : Calculer le hash (si fichier extrait)
```bash
# Linux / macOS
md5sum update.exe
sha256sum update.exe

# Windows (PowerShell)
Get-FileHash update.exe -Algorithm MD5
Get-FileHash update.exe -Algorithm SHA256
```

**🔍 Rechercher le hash sur VirusTotal.com**

#### Étape 7 : Détecter le callback C2
```
Filtre : ip.src == 192.168.1.75 && tcp.port == 443
```

**🔍 Cherchez une connexion après le téléchargement :**
- Vers une IP externe (203.0.113.200)
- ~10 secondes après le téléchargement
- Connexion HTTPS (chiffrée)

**✅ C'est le malware qui contacte son serveur C2**

---

## 🎯 LAB 7 : SQL Injection Attack

### 📋 Objectifs
- Identifier les payloads SQL Injection
- Déterminer quelles tentatives ont réussi
- Extraire les données exposées

### 🔍 Instructions d'Analyse

#### Étape 1 : Filtrer les requêtes HTTP
```
Filtre : http
```

#### Étape 2 : Rechercher les mots-clés SQL
```
Filtre : http.request.uri contains "SELECT" || 
         http.request.uri contains "UNION" || 
         http.request.uri contains "DROP" ||
         http.request.uri contains "' OR '"
```

**📋 Ce que vous devriez voir :**
- Multiples tentatives avec syntaxe SQL
- Toutes depuis la même IP (203.0.113.75)

#### Étape 3 : Analyser chaque tentative

**Méthode manuelle :**
1. Sélectionner une requête HTTP avec SQL
2. Clic droit → Follow → HTTP Stream
3. Lire la requête ET la réponse

**Exemples de payloads :**

```
❌ ÉCHEC :
GET /login.php?user=admin'--&pass=anything
→ HTTP/1.1 200 OK
→ <html>Login failed</html>

✅ SUCCÈS :
GET /login.php?user=admin' OR '1'='1&pass=test
→ HTTP/1.1 200 OK
→ User: admin, Password: 5f4dcc3b5aa765d61d8327deb882cf99

⚠️ ERREUR SQL EXPOSÉE :
GET /products.php?id=1'; DROP TABLE users--
→ HTTP/1.1 500 Internal Server Error
→ MySQL Error: You have an error in your SQL syntax...
```

#### Étape 4 : Identifier l'outil utilisé
```
Filtre : http.user_agent
```

**🔍 Dans les détails du paquet :**
```
User-Agent: sqlmap/1.6.12
```

**💡 SQLMap = outil d'exploitation SQL automatisé**

#### Étape 5 : Extraction des données compromises

**Rechercher les réponses HTTP avec données sensibles :**
```
Filtre : http && (http contains "Password" || http contains "username")
```

**📋 Données exposées dans ce LAB :**
```
Username: admin
Password Hash: 5f4dcc3b5aa765d61d8327deb882cf99
Email: admin@company.com
```

**🔓 Cracker le hash (MD5) :**
```bash
# Utiliser un site comme crackstation.net
# Ou John the Ripper / hashcat

Hash : 5f4dcc3b5aa765d61d8327deb882cf99
Resultat : password (mot de passe faible!)
```

---

## 🎯 LAB 8 : ARP Spoofing (MITM)

### 📋 Objectifs
- Détecter l'empoisonnement ARP
- Identifier l'attaquant
- Voir les données interceptées

### 🔍 Instructions d'Analyse

#### Étape 1 : Filtrer le trafic ARP
```
Filtre : arp
```

#### Étape 2 : Analyser les réponses ARP
```
Menu : Statistics → Endpoints → Ethernet
```

**🔍 Recherchez :**
- Multiples adresses MAC
- Noter particulièrement les MAC pour `192.168.1.1` (gateway)

#### Étape 3 : Détecter les duplications
```
Filtre : arp.opcode == 2
```

**ARP opcode 2 = ARP Reply (réponse)**

**🚨 Indicateur de spoofing :**
Deux paquets avec :
- Même IP source (`192.168.1.1`)
- MAC source différente

**Exemple :**
```
Paquet 1 : 192.168.1.1 is at 00:50:56:ff:ee:dd (légitime)
Paquet 2 : 192.168.1.1 is at 00:0c:29:aa:bb:cc (malveillant)
```

#### Étape 4 : Identifier l'attaquant
```
Filtre : arp.src.proto_ipv4 == 192.168.1.1 && arp.src.hw_mac == 00:0c:29:aa:bb:cc
```

**💡 Logique :**
- Qui prétend être la gateway (`192.168.1.1`)
- Mais avec une MAC différente
- **C'est l'attaquant : 00:0c:29:aa:bb:cc**

**Résoudre l'IP de l'attaquant :**
```
Filtre : eth.src == 00:0c:29:aa:bb:cc
```

**✅ Réponse : 192.168.1.66**

#### Étape 5 : Visualiser le trafic intercepté
```
Filtre : http && ip.src == 192.168.1.75
```

**🔍 Suivre le flux HTTP :**
```
Sélectionner un paquet HTTP
Clic droit → Follow → HTTP Stream
```

**🔓 Données en clair interceptées :**
```
GET /login HTTP/1.1
Authorization: Basic YWRtaW46cGFzc3dvcmQxMjM=
```

**Décoder l'Authorization header :**
```bash
echo "YWRtaW46cGFzc3dvcmQxMjM=" | base64 -d
```

**✅ Résultat : `admin:password123`**

**🚨 L'attaquant a capturé les credentials !**

#### Étape 6 : Tracer le flux des paquets

**Observation du MITM :**
```
Victime (192.168.1.75) 
    → envoie vers MAC attaquant (pensant que c'est la gateway)
    
Attaquant (192.168.1.66)
    → forward vers vraie gateway
    → peut lire/modifier le trafic
    
Gateway
    → répond normalement
    
Attaquant
    → forward la réponse à la victime
```

---

## 🏆 Exercice Final : Analyse Multi-Scénario

### Mission
Vous recevez un fichier PCAP d'un incident réel. Utilisez TOUS les LABs pour :

1. **Identifier le vecteur d'attaque initial**
   - Scan ? Phishing ? Exploit ?

2. **Tracer la compromission**
   - Malware download ? Brute force réussi ?

3. **Détecter la persistance**
   - C2 beacon ? Backdoor ?

4. **Repérer le mouvement latéral**
   - SMB ? RDP ? WMI ?

5. **Identifier l'exfiltration**
   - DNS tunneling ? HTTPS ? FTP ?

### Checklist d'Investigation

```
□ Statistics → Protocol Hierarchy (vue d'ensemble)
□ Statistics → Conversations (IPs suspectes)
□ Statistics → DNS (domaines anormaux)
□ Filtre : http (téléchargements, requêtes)
□ Filtre : tcp.port == 22/3389/445 (services critiques)
□ Filtre : tcp.port == 443 && io_graph (beacons)
□ Recherche : mots-clés (SELECT, UNION, admin', etc.)
□ Export Objects → HTTP (fichiers téléchargés)
□ Resolved Addresses (MAC/IP mapping)
```

---

## 📚 Ressources Complémentaires

### Sites pour pratiquer
- **Malware-Traffic-Analysis.net** - PCAPs réels d'infections
- **NetreSec.com** - Captures réseau variées
- **CyberDefenders.org** - Challenges Blue Team
- **TryHackMe** - Rooms Wireshark interactives

### Documentation
- **Wireshark User Guide** : https://www.wireshark.org/docs/wsug_html_chunked/
- **Display Filters Reference** : https://www.wireshark.org/docs/dfref/
- **Sample Captures** : https://wiki.wireshark.org/SampleCaptures

### Outils complémentaires
- **NetworkMiner** - Extraction automatique d'artefacts
- **Zeek (Bro)** - Analyse réseau scriptable
- **Suricata** - IDS/IPS avec logs détaillés
- **tcpdump** - Capture en ligne de commande

---

## 🎓 Certification et Formation

### Wireshark Certified Network Analyst (WCNA)
- Formation officielle Wireshark
- Certification reconnue
- https://www.wiresharktraining.com

### Cours recommandés
- **Chris Greer** - YouTube (excellent pédagogue)
- **David Bombal** - Cours networking/security
- **Cyber Mentor** - SOC Analyst training

---

## ✅ Solutions des LABs

### LAB 1 - Port Scan
- **Attaquant** : 192.168.1.50
- **Type de scan** : SYN Scan (stealth)
- **Ports ouverts** : 22, 80, 443, 3306
- **Vulnérabilité critique** : MySQL (3306) exposé

### LAB 2 - SSH Brute Force
- **Attaquant** : 203.0.113.50
- **Tentatives** : 500
- **Compromission** : OUI (connexion #458, durée 847s)
- **Action requise** : Isoler la machine, changer tous les mots de passe

### LAB 3 - DNS Tunneling
- **Victime** : 192.168.1.75
- **Domaine C2** : strange-domain.xyz
- **Serveur DNS C2** : 203.0.113.100
- **Données exfiltrées** : Rapport financier Q1 2024
- **Volume** : ~45 Ko

### LAB 4 - C2 Beacon
- **Victime** : 192.168.1.75
- **Serveur C2** : 203.0.113.200
- **Intervalle beacon** : 60 secondes
- **Nombre de beacons** : 30 (sur 30 minutes)

### LAB 5 - Lateral Movement
- **Machine compromise** : 192.168.1.50
- **Cibles** : .101, .102, .103, .104
- **Méthode** : PsExec via SMB
- **Partages** : ADMIN$, C$

### LAB 6 - Malware Download
- **Victime** : 192.168.1.75
- **Site compromis** : malicious-blog.com
- **Serveur malware** : 203.0.113.150
- **Fichier** : update.exe (fake Windows update)
- **Callback C2** : 203.0.113.200 (10s après download)

### LAB 7 - SQL Injection
- **Attaquant** : 203.0.113.75
- **Cible** : 192.168.1.200 (webapp)
- **Outil** : SQLMap 1.6.12
- **Succès** : 3 payloads réussis
- **Données exposées** : admin credentials (MD5 hash)
- **Password cracké** : password

### LAB 8 - ARP Spoofing
- **Attaquant** : 192.168.1.66 (MAC: 00:0c:29:aa:bb:cc)
- **Victime** : 192.168.1.75
- **Gateway usurpée** : 192.168.1.1
- **Données interceptées** : HTTP credentials (admin:password123)
- **Méthode de détection** : Duplicate ARP replies

---

## 🚨 Rappel Important

Ces fichiers PCAP sont des **simulations pédagogiques**.
- Ne contiennent AUCUNE donnée réelle ou sensible
- Générés avec Scapy pour apprentissage
- Utilisables librement dans un cadre éducatif

**Bon apprentissage et excellente analyse ! 🔍**

---

## 🛡️ Règles de Détection Wazuh

Un fichier complet avec **toutes les règles Wazuh** pour détecter automatiquement ces attaques en production a été créé :

📄 **WAZUH_DETECTION_RULES.md**

Ce fichier contient :
- ✅ Règles XML prêtes à l'emploi pour chaque LAB
- ✅ Configuration des décodeurs personnalisés
- ✅ Scripts Python d'analyse avancée
- ✅ Active Response pour blocage automatique
- ✅ Intégration MITRE ATT&CK
- ✅ Configuration des dashboards
- ✅ Gestion des faux positifs

### 📋 Résumé des Règles par LAB

| LAB | Nombre de Règles | Niveau Max | Techniques MITRE |
|-----|------------------|------------|------------------|
| **LAB 1** - Port Scan | 5 règles | 10 | T1046 |
| **LAB 2** - SSH Brute Force | 6 règles | 14 | T1110.001 |
| **LAB 3** - DNS Tunneling | 6 règles | 10 | T1048.003 |
| **LAB 4** - C2 Beacon | 6 règles | 11 | T1071.001 |
| **LAB 5** - Lateral Movement | 7 règles | 12 | T1021.002 |
| **LAB 6** - Malware Download | 7 règles | 13 | T1204.002 |
| **LAB 7** - SQL Injection | 9 règles | 13 | T1190 |
| **LAB 8** - ARP Spoofing | 6 règles | 14 | T1557.002 |

**Total : 52 règles de détection**

### 🚀 Déploiement Rapide

```bash
# 1. Télécharger le fichier WAZUH_DETECTION_RULES.md
# 2. Extraire les règles XML
# 3. Les ajouter à /var/ossec/etc/rules/local_rules.xml

sudo nano /var/ossec/etc/rules/local_rules.xml

# 4. Redémarrer Wazuh
sudo systemctl restart wazuh-manager

# 5. Vérifier les règles
/var/ossec/bin/wazuh-logtest
```

### 💡 Avantage Pédagogique

En combinant :
- **Wireshark** → Analyse forensique post-incident
- **Wazuh** → Détection en temps réel

Vous développez une **vision complète** de la détection des menaces :
- 🔍 Comment analyser après coup
- 🛡️ Comment détecter en temps réel
- ⚡ Comment réagir automatiquement

---

