# 🎯 LAB 1 : Détection de Scan de Ports (Reconnaissance)

<div align="center">

![Difficulty](https://img.shields.io/badge/Difficulté-Débutant-green?style=for-the-badge)
![MITRE](https://img.shields.io/badge/MITRE-T1046-red?style=for-the-badge)
![Duration](https://img.shields.io/badge/Durée-10--15_min-blue?style=for-the-badge)

**Scénario réaliste de détection de reconnaissance réseau avec Nmap**

[📥 Télécharger le PCAP](../pcaps/LAB1_Port_Scan.pcap) • [🛡️ Règles Wazuh](#règles-de-détection-wazuh)

</div>

---

## 📖 Contexte Théorique

### Qu'est-ce qu'un scan de ports ?

Un **scan de ports** est la première phase d'une attaque : la **reconnaissance**.

L'attaquant cherche à identifier :
- ✅ Quels services sont actifs sur la cible
- ✅ Quels ports sont ouverts
- ✅ Quelle est la version des services
- ✅ Quel système d'exploitation est utilisé

> 💡 **Analogie** : C'est comme "frapper à toutes les portes d'un immeuble" pour voir lesquelles s'ouvrent.

### Pourquoi c'est dangereux ?

| Risque | Description |
|--------|-------------|
| **Cartographie** | L'attaquant cartographie votre infrastructure |
| **Vulnérabilités** | Identifie les services potentiellement vulnérables |
| **Précurseur** | Toujours suivi d'une exploitation si des failles sont trouvées |
| **Intelligence** | Collecte d'informations pour une attaque ciblée |

### Techniques de Scan

#### 🔍 Types de Scans TCP

| Type | Flags TCP | Furtivité | Description |
|------|-----------|-----------|-------------|
| **SYN Scan** | `SYN` | ⭐⭐⭐ Élevée | Scan furtif (stealth), ne complète pas le handshake |
| **Connect Scan** | `SYN, ACK, RST` | ⭐ Faible | Connexion complète, très visible |
| **FIN Scan** | `FIN` | ⭐⭐⭐⭐ Très élevée | Envoie FIN, les ports ouverts ne répondent pas |
| **XMAS Scan** | `FIN, PSH, URG` | ⭐⭐⭐⭐ Très élevée | "Sapin de Noël", tous les flags activés |
| **NULL Scan** | Aucun | ⭐⭐⭐⭐ Très élevée | Aucun flag, les ports ouverts ne répondent pas |

---

## 🎬 Scénario du LAB

### Contexte de l'Incident

**Date** : 20 mai 2024, 14:23:01  
**Alerte SIEM** : "Activité de scan potentielle détectée depuis 192.168.1.50"

```
╔══════════════════════════════════════════════════════════════╗
║                    ALERTE SÉCURITÉ                           ║
╠══════════════════════════════════════════════════════════════╣
║ Type         : Port Scan Detection                           ║
║ Sévérité     : MOYENNE                                       ║
║ Source       : 192.168.1.50                                  ║
║ Destination  : 192.168.1.100 (srv-web-prod)                 ║
║ Timestamp    : 2024-05-20 14:23:01                          ║
║                                                               ║
║ Description  : Multiples tentatives de connexion TCP         ║
║                détectées vers différents ports               ║
╚══════════════════════════════════════════════════════════════╝
```

### Votre Mission

En tant qu'analyste SOC, vous devez :

1. ✅ **Confirmer** qu'il s'agit bien d'un scan de ports
2. ✅ **Identifier** le type de scan utilisé
3. ✅ **Découvrir** quels ports sont ouverts
4. ✅ **Déterminer** si l'attaquant a découvert des vulnérabilités
5. ✅ **Documenter** pour blocage firewall

---

## 🔍 Investigation Wireshark - Pas-à-Pas

### Étape 1 : Ouvrir la Capture

```bash
# Télécharger le fichier PCAP
wget https://github.com/VOTRE_USERNAME/wireshark-labs-soc-analyst/raw/main/pcaps/LAB1_Port_Scan.pcap

# Ouvrir avec Wireshark
wireshark LAB1_Port_Scan.pcap
```

**📊 Informations sur la capture :**
- **Nombre de paquets** : 2,006
- **Taille** : 138 KB
- **Durée** : ~1 seconde
- **Protocole principal** : TCP

---

### Étape 2 : Premier Filtre - Détecter le Scan SYN

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

#### 💡 Explication du Filtre

| Composant | Signification |
|-----------|---------------|
| `tcp.flags.syn == 1` | Le flag SYN est activé |
| `tcp.flags.ack == 0` | Le flag ACK n'est PAS activé |
| **Combiné** | = Paquets SYN sans ACK = **Initiation de connexion** |

#### 📊 Ce que vous devriez voir

```
No.   Time        Source         Destination    Protocol  Info
1234  14:23:01.1  192.168.1.50   192.168.1.100  TCP       54321 → 21 [SYN]
1235  14:23:01.1  192.168.1.50   192.168.1.100  TCP       54322 → 22 [SYN]
1236  14:23:01.1  192.168.1.50   192.168.1.100  TCP       54323 → 23 [SYN]
1237  14:23:01.1  192.168.1.50   192.168.1.100  TCP       54324 → 80 [SYN]
1238  14:23:01.1  192.168.1.50   192.168.1.100  TCP       54325 → 443 [SYN]
... (des centaines de lignes similaires)
```

#### 🚨 Pourquoi c'est Suspect ?

| Observation | Explication |
|-------------|-------------|
| **Même source** | 192.168.1.50 pour tous les paquets |
| **Même destination** | 192.168.1.100 pour tous les paquets |
| **Ports différents** | 21, 22, 23, 80, 443... tous les ports |
| **Fréquence élevée** | Plusieurs paquets par milliseconde |
| **Pattern automatisé** | Aucun humain ne peut faire ça manuellement |

> 🎯 **Conclusion** : C'est un **scan automatisé** (probablement Nmap)

---

### Étape 3 : Analyser les Conversations TCP

```
Menu : Statistics → Conversations → Onglet TCP
```

#### 💡 Ce Menu Fait Quoi ?

Ce menu liste **toutes les conversations TCP** dans la capture :
- Une conversation = échange entre 2 IPs sur un port spécifique
- Permet de voir rapidement les patterns anormaux
- Tri par nombre de paquets, durée, taille, etc.

#### 📊 Résultat Typique d'un Scan

```
Address A         Port A  <->  Address B         Port B  Packets  Bytes  Duration
192.168.1.50      54321        192.168.1.100     21      2        120    0.002s
192.168.1.50      54322        192.168.1.100     22      4        240    0.005s ✅
192.168.1.50      54323        192.168.1.100     23      2        120    0.002s
192.168.1.50      54324        192.168.1.100     80      4        240    0.005s ✅
192.168.1.50      54325        192.168.1.100     443     4        240    0.005s ✅
192.168.1.50      54326        192.168.1.100     3306    4        240    0.005s ✅
... (1000 lignes similaires)
```

#### 🔎 Analyse des Résultats

| Paquets | Interprétation | Signification |
|---------|----------------|---------------|
| **2 paquets** | `SYN → (silence)` | Port **fermé** ou filtré |
| **4 paquets** | `SYN → SYN-ACK → ACK → RST` | Port **OUVERT** ✅ |

#### 💡 Pourquoi 4 Paquets pour un Port Ouvert ?

```
1. Attaquant → Cible : [SYN]        "Je veux me connecter au port 22"
2. Cible → Attaquant : [SYN-ACK]    "OK, connectons-nous"
3. Attaquant → Cible : [ACK]        "Connexion confirmée"
4. Attaquant → Cible : [RST]        "En fait, je ferme immédiatement"
                                     ↑ Typique d'un scan (pas une vraie connexion)
```

---

### Étape 4 : Identifier les Ports Ouverts Découverts

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 1 && ip.src == 192.168.1.100
```

#### 💡 Explication du Filtre

| Composant | Signification |
|-----------|---------------|
| `tcp.flags.syn == 1` | Flag SYN activé |
| `tcp.flags.ack == 1` | Flag ACK activé |
| `ip.src == 192.168.1.100` | Provenant de la **cible** (pas l'attaquant) |
| **Combiné** | = Réponses SYN-ACK = **Ports ouverts** |

#### 📊 Résultats - Ports Ouverts Détectés

```
No.   Source         Destination    Info                         Port
1456  192.168.1.100  192.168.1.50   22 → 54322 [SYN, ACK]       SSH ✅
1789  192.168.1.100  192.168.1.50   80 → 54380 [SYN, ACK]       HTTP ✅
2034  192.168.1.100  192.168.1.50   443 → 54443 [SYN, ACK]      HTTPS ✅
2567  192.168.1.100  192.168.1.50   3306 → 54521 [SYN, ACK]     MySQL ⚠️
```

#### 🚨 ALERTE CRITIQUE : Port MySQL Exposé

| Port | Service | Légitime ? | Risque |
|------|---------|------------|--------|
| 22 | SSH | ✅ Oui | Accès admin nécessaire |
| 80 | HTTP | ✅ Oui | Serveur web public |
| 443 | HTTPS | ✅ Oui | Serveur web sécurisé |
| **3306** | **MySQL** | ❌ **NON** | **Base de données NE DEVRAIT PAS être exposée** |

> 🚨 **Vulnérabilité Majeure** : MySQL (port 3306) est accessible depuis le réseau. Cela permet potentiellement :
> - Attaques par brute force sur le mot de passe root
> - Exploitation de vulnérabilités MySQL
> - Accès direct aux données sensibles

---

### Étape 5 : Créer une Timeline de l'Attaque

```
Menu : Statistics → IO Graph
```

#### Configuration du Graphique

1. **Filter** : `tcp.flags.syn == 1 && ip.src == 192.168.1.50`
2. **Y-Axis** : COUNT
3. **Interval** : 1 second
4. **Cliquer sur "Graph"**

#### 📊 Graphique Résultant

```
Paquets SYN/seconde
    |
1000|     ████████████
    |    ██████████████
 800|   ████████████████
    |  ██████████████████
 600| ████████████████████
    |██████████████████████
 400|████████████████████████
    |████████████████████████
 200|████████████████████████
    |████████████████████████
   0|_________________________
     14:23:00  14:23:20  14:23:40  14:24:00
          ↑           ↑
        Début       Fin
```

#### 💡 Interprétation de la Timeline

| Observation | Signification |
|-------------|---------------|
| **Pic soudain** | L'attaque commence brutalement |
| **Plateau élevé** | Scan en cours (~1000 paquets/seconde) |
| **Chute brusque** | Fin du scan (l'attaquant a terminé) |
| **Durée totale** | ~1 seconde (scan **TRÈS rapide**) |

#### 🚨 Comparaison avec Trafic Normal

| Type de Trafic | Paquets/seconde |
|----------------|-----------------|
| Navigation web normale | 5-50 |
| Téléchargement fichier | 100-500 |
| **Scan de ports** | **1000+** 🚨 |

> 🎯 **Conclusion** : Vitesse anormale = outil automatisé professionnel (Nmap avec `-T4` ou `-T5`)

---

### Étape 6 : Identifier le Type de Scan Nmap

#### Analyser les Flags TCP en Détail

```
Sélectionner un paquet SYN → Développer "Transmission Control Protocol"
```

**Observation des flags :**

```
Flags: 0x002 (SYN)
    000. .... .... = Reserved: Not set
    ...0 .... .... = Nonce: Not set
    .... 0... .... = Congestion Window Reduced (CWR): Not set
    .... .0.. .... = ECN-Echo: Not set
    .... ..0. .... = Urgent: Not set
    .... ...0 .... = Acknowledgment: Not set
    .... .... 0... = Push: Not set
    .... .... .0.. = Reset: Not set
    .... .... ..1. = Syn: Set ✅
    .... .... ...0 = Fin: Not set
```

#### 🎯 Identification : SYN Scan (Stealth Scan)

| Caractéristique | Observation dans le PCAP |
|-----------------|--------------------------|
| **Flags** | Uniquement SYN (pas ACK, pas FIN, pas PSH) |
| **Handshake** | Incomplet (SYN → SYN-ACK → RST) |
| **Furtivité** | ⭐⭐⭐ Élevée |
| **Outil** | Nmap par défaut (`nmap -sS`) |

#### Commande Nmap Probable

```bash
# L'attaquant a probablement utilisé :
nmap -sS -p 1-1000 192.168.1.100

# Options :
# -sS     : SYN scan (stealth)
# -p 1-1000 : Scanner les ports 1 à 1000
# 192.168.1.100 : Cible
```

---

## 📊 Statistiques de la Capture

### Résumé des Findings

| Métrique | Valeur |
|----------|--------|
| **Paquets totaux** | 2,006 |
| **IP Attaquante** | 192.168.1.50 |
| **IP Victime** | 192.168.1.100 (srv-web-prod) |
| **Type de scan** | SYN Scan (Stealth) |
| **Ports scannés** | 1-1000 (1000 ports) |
| **Ports ouverts** | 4 (SSH, HTTP, HTTPS, MySQL) |
| **Durée totale** | ~1 seconde |
| **Vitesse** | ~1000 paquets/seconde |
| **Outil probable** | Nmap `-sS` `-T4` ou `-T5` |

---

## 🎯 Extraction des IOCs (Indicators of Compromise)

### Indicateurs à Documenter

```yaml
Incident: Port Scan Detection
Date: 2024-05-20 14:23:01 UTC

Attacker:
  IP: 192.168.1.50
  MAC: [à extraire si disponible]
  Hostname: [inconnu]
  Location: [à géolocaliser]

Target:
  IP: 192.168.1.100
  Hostname: srv-web-prod
  Services_Discovered:
    - Port: 22, Service: SSH, Status: Open
    - Port: 80, Service: HTTP, Status: Open
    - Port: 443, Service: HTTPS, Status: Open
    - Port: 3306, Service: MySQL, Status: Open, Risk: CRITICAL

Attack_Details:
  Type: Port Scan
  Method: SYN Scan (Stealth)
  Tool: Nmap (probable)
  Ports_Scanned: 1-1000
  Duration: 1 second
  Packets_Per_Second: 1000

MITRE_ATTCK:
  Tactic: Reconnaissance
  Technique: T1046 - Network Service Discovery
  Sub_Technique: T1046.001 - Port Scanning
```

---

## 📋 Rapport d'Incident Professionnel

```
╔══════════════════════════════════════════════════════════════╗
║          RAPPORT D'INCIDENT - SCAN DE PORTS                  ║
╠══════════════════════════════════════════════════════════════╣
║ ID Incident    : SOC-2024-0342                               ║
║ Date/Heure     : 2024-05-20 14:23:01 UTC                     ║
║ Analyste       : [Votre nom]                                 ║
║ Sévérité       : MOYENNE → ÉLEVÉE                            ║
╠══════════════════════════════════════════════════════════════╣
║ RÉSUMÉ EXÉCUTIF                                               ║
╠══════════════════════════════════════════════════════════════╣
║ Un scan de ports automatisé a été détecté ciblant le         ║
║ serveur web de production (srv-web-prod). L'attaquant        ║
║ a utilisé un scan SYN furtif pour cartographier les          ║
║ services exposés. Une vulnérabilité critique a été            ║
║ identifiée : MySQL (port 3306) est accessible depuis         ║
║ le réseau, ce qui ne devrait PAS être le cas.                ║
╠══════════════════════════════════════════════════════════════╣
║ INDICATEURS DE COMPROMISSION (IOCs)                          ║
╠══════════════════════════════════════════════════════════════╣
║ IP Attaquante  : 192.168.1.50                                ║
║ IP Victime     : 192.168.1.100 (srv-web-prod)                ║
║ Type d'attaque : SYN Scan (Stealth)                          ║
║ Outil probable : Nmap                                        ║
║ Durée          : 1 seconde                                   ║
║ Ports scannés  : 1-1000 (scan complet)                       ║
╠══════════════════════════════════════════════════════════════╣
║ PORTS OUVERTS DÉCOUVERTS PAR L'ATTAQUANT                     ║
╠══════════════════════════════════════════════════════════════╣
║ • 22/tcp   (SSH)     - ✅ Légitime (accès admin)             ║
║ • 80/tcp   (HTTP)    - ✅ Légitime (serveur web)             ║
║ • 443/tcp  (HTTPS)   - ✅ Légitime (serveur web SSL)         ║
║ • 3306/tcp (MySQL)   - ⚠️  VULNÉRABILITÉ CRITIQUE            ║
║                        NE DEVRAIT PAS ÊTRE EXPOSÉ            ║
╠══════════════════════════════════════════════════════════════╣
║ ANALYSE TECHNIQUE                                             ║
╠══════════════════════════════════════════════════════════════╣
║ Phase d'attaque : RECONNAISSANCE (Cyber Kill Chain)          ║
║                                                               ║
║ L'attaquant a utilisé un scan SYN (stealth) pour             ║
║ identifier les services actifs. La vitesse d'exécution       ║
║ (~1000 paquets/seconde) indique l'utilisation d'un outil     ║
║ professionnel (Nmap avec options -T4 ou -T5).                ║
║                                                               ║
║ La découverte du port MySQL (3306) ouvert représente une     ║
║ vulnérabilité critique. Ce port ne devrait être accessible   ║
║ que depuis le réseau local backend, jamais depuis le         ║
║ réseau principal.                                             ║
╠══════════════════════════════════════════════════════════════╣
║ RISQUES IDENTIFIÉS                                            ║
╠══════════════════════════════════════════════════════════════╣
║ 🔴 CRITIQUE - Port MySQL exposé :                            ║
║    • Attaque par brute force du mot de passe root            ║
║    • Exploitation de vulnérabilités MySQL (CVE)              ║
║    • Accès direct aux données sensibles                      ║
║    • Injection SQL potentielle                               ║
║                                                               ║
║ 🟡 MOYEN - Scan réussi :                                     ║
║    • Cartographie complète obtenue par l'attaquant           ║
║    • Préparation probable d'attaque ciblée                   ║
║    • Tentative de brute force SSH imminente                  ║
╠══════════════════════════════════════════════════════════════╣
║ ACTIONS RECOMMANDÉES (PAR PRIORITÉ)                          ║
╠══════════════════════════════════════════════════════════════╣
║ ✅ IMMÉDIAT (0-1h)                                            ║
║   • Bloquer 192.168.1.50 sur le firewall                     ║
║   • Vérifier si 192.168.1.50 est légitime (IP interne)      ║
║   • Activer monitoring renforcé sur srv-web-prod             ║
║                                                               ║
║ ⚠️ URGENT (1-4h)                                              ║
║   • FERMER le port 3306 ou restreindre aux IPs backend      ║
║   • Audit des règles firewall MySQL                          ║
║   • Vérifier les logs MySQL pour tentatives d'accès          ║
║                                                               ║
║ 🔧 COURT TERME (1-24h)                                        ║
║   • Renforcer authentification SSH (clés + 2FA)              ║
║   • Déployer IPS avec règles anti-scan                       ║
║   • Analyser logs pour activité post-scan                    ║
║   • Notification équipe sécurité                             ║
║                                                               ║
║ 🛡️ MOYEN TERME (1-7 jours)                                   ║
║   • Audit complet de segmentation réseau                     ║
║   • Déploiement de règles Wazuh pour détection scan         ║
║   • Pentest interne pour validation sécurité                 ║
║   • Formation équipe sur détection reconnaissance            ║
╠══════════════════════════════════════════════════════════════╣
║ PREUVES FORENSIQUES                                           ║
╠══════════════════════════════════════════════════════════════╣
║ • LAB1_Port_Scan.pcap (capture Wireshark complète)          ║
║ • timeline_io_graph.png (graphique d'attaque)                ║
║ • open_ports_list.txt (liste des ports découverts)           ║
║ • firewall_logs_excerpt.log (logs corrélés)                  ║
╠══════════════════════════════════════════════════════════════╣
║ MITRE ATT&CK MAPPING                                          ║
╠══════════════════════════════════════════════════════════════╣
║ Tactic      : Reconnaissance                                 ║
║ Technique   : T1046 - Network Service Discovery              ║
║ Sub-Technique: T1046.001 - Port Scanning                     ║
║ Procedure   : Nmap SYN scan for service enumeration          ║
╚══════════════════════════════════════════════════════════════╝
```

---

## 🛡️ Règles de Détection Wazuh

### Configuration Complète pour Détection Automatique

#### Règles XML à Ajouter dans `/var/ossec/etc/rules/local_rules.xml`

```xml
<!-- ═══════════════════════════════════════════════════════════════ -->
<!-- LAB 1 : Détection de Scan de Ports                              -->
<!-- ═══════════════════════════════════════════════════════════════ -->

<group name="lab1,port_scan,reconnaissance,">

  <!-- Règle 1: Détection de scan de ports basique -->
  <rule id="100001" level="8">
    <if_group>firewall</if_group>
    <description>Port scan détecté - Multiples ports ciblés depuis une même source</description>
    <mitre>
      <id>T1046</id>
    </mitre>
  </rule>

  <!-- Règle 2: Scan de ports intensif (Nmap agressif) -->
  <rule id="100002" level="10" frequency="50" timeframe="60">
    <if_matched_sid>100001</if_matched_sid>
    <description>Port scan intensif détecté - Plus de 50 tentatives en 60 secondes</description>
    <group>intrusion_attempt,pci_dss_11.4,gdpr_IV_35.7.d,</group>
    <mitre>
      <id>T1046</id>
    </mitre>
  </rule>

  <!-- Règle 3: Scan SYN (stealth scan) -->
  <rule id="100003" level="9">
    <if_group>ids</if_group>
    <match>SYN scan|stealth scan|half-open scan</match>
    <description>Scan SYN (stealth) détecté - Tentative de scan furtif</description>
    <mitre>
      <id>T1046</id>
    </mitre>
  </rule>

  <!-- Règle 4: Scan de ports critiques (SSH, RDP, SMB, MySQL) -->
  <rule id="100004" level="10" frequency="5" timeframe="30">
    <if_group>firewall</if_group>
    <match>port 22|port 3389|port 445|port 3306|port 1433</match>
    <description>Scan ciblé sur ports critiques détecté</description>
    <group>service_availability,pci_dss_10.6.1,</group>
    <mitre>
      <id>T1046</id>
    </mitre>
  </rule>

  <!-- Règle 5: Whitelist pour scanners autorisés -->
  <rule id="100005" level="0">
    <if_sid>100001,100002,100003,100004</if_sid>
    <srcip>192.168.1.250</srcip> <!-- IP du scanner Nessus autorisé -->
    <description>Port scan depuis scanner autorisé - Ignoré</description>
  </rule>

</group>
```

### Active Response (Blocage Automatique)

Ajouter dans `/var/ossec/etc/ossec.conf` :

```xml
<!-- Bloquer automatiquement l'IP après détection de scan intensif -->
<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>100002</rules_id>
  <timeout>1800</timeout> <!-- Bloquer pendant 30 minutes -->
</active-response>
```

### Test des Règles

```bash
# Tester avec wazuh-logtest
sudo /var/ossec/bin/wazuh-logtest

# Exemple de log à tester :
# Jan 20 14:23:01 firewall kernel: [UFW BLOCK] IN=eth0 SRC=192.168.1.50 DST=192.168.1.100 PROTO=TCP SPT=50001 DPT=22 WINDOW=1024 SYN
```

### Dashboard Wazuh Recommandé

```json
{
  "title": "Port Scan Detection - LAB 1",
  "panels": [
    {
      "title": "Port Scans par Source IP",
      "type": "pie",
      "query": "rule.id:100002"
    },
    {
      "title": "Timeline des Scans",
      "type": "timeline",
      "query": "rule.groups:port_scan"
    },
    {
      "title": "Top Ports Ciblés",
      "type": "bar",
      "query": "rule.id:100004"
    }
  ]
}
```

---

## 📚 Ressources Complémentaires

### Documentation Nmap

- **Site officiel** : https://nmap.org/
- **Guide de référence** : https://nmap.org/book/man.html
- **Types de scans** : https://nmap.org/book/man-port-scanning-techniques.html

### Techniques de Détection

- **Wireshark Display Filters** : https://wiki.wireshark.org/DisplayFilters
- **TCP Flags** : https://www.wireshark.org/docs/dfref/t/tcp.html
- **MITRE ATT&CK T1046** : https://attack.mitre.org/techniques/T1046/

### Outils de Défense

- **PortSentry** : Détection active de scans
- **psad** : Port Scan Attack Detector
- **Fail2Ban** : Bannissement automatique
- **Suricata** : IDS/IPS avec règles anti-scan

---

## ✅ Checklist de Validation

### Compétences à Valider

Après ce LAB, vous devriez être capable de :

```
□ Identifier un scan de ports dans une capture PCAP
□ Différencier un SYN scan d'un Connect scan
□ Utiliser les filtres Wireshark efficacement
□ Analyser les statistiques de conversations TCP
□ Créer une timeline d'attaque avec IO Graph
□ Extraire les ports ouverts découverts
□ Identifier l'outil utilisé (Nmap)
□ Rédiger un rapport d'incident professionnel
□ Configurer des règles Wazuh de détection
□ Mapper l'attaque au framework MITRE ATT&CK
```

### Questions de Validation

1. **Quel type de scan a été utilisé dans ce LAB ?**
   <details>
   <summary>Voir la réponse</summary>
   SYN Scan (Stealth Scan) - détecté par les flags TCP SYN sans ACK
   </details>

2. **Quels ports sont ouverts sur la cible ?**
   <details>
   <summary>Voir la réponse</summary>
   22 (SSH), 80 (HTTP), 443 (HTTPS), 3306 (MySQL)
   </details>

3. **Quelle est la vulnérabilité critique identifiée ?**
   <details>
   <summary>Voir la réponse</summary>
   Port MySQL (3306) exposé publiquement - ne devrait être accessible que depuis le backend
   </details>

4. **Quelle commande Nmap l'attaquant a-t-il probablement utilisée ?**
   <details>
   <summary>Voir la réponse</summary>
   `nmap -sS -p 1-1000 192.168.1.100`
   </details>

---

## 🎓 Prochaines Étapes

### LAB Suivant : LAB 2 - SSH Brute Force

Maintenant que vous savez détecter la reconnaissance, apprenez à identifier une **attaque par force brute** et une **compromission réussie**.

[➡️ Aller au LAB 2](LAB2_SSH_BRUTEFORCE_DETECTION.md)

---

## 🤝 Contribution

Vous avez des suggestions d'amélioration pour ce LAB ?

- 🐛 **Bug** : Ouvrir une [Issue](https://github.com/VOTRE_USERNAME/wireshark-labs-soc-analyst/issues)
- 💡 **Amélioration** : Proposer une [Pull Request](https://github.com/VOTRE_USERNAME/wireshark-labs-soc-analyst/pulls)
- 💬 **Discussion** : Rejoindre [Discussions](https://github.com/VOTRE_USERNAME/wireshark-labs-soc-analyst/discussions)

---

<div align="center">

**Félicitations pour avoir complété le LAB 1 ! 🎉**

[![Retour au README](https://img.shields.io/badge/←_Retour_au_README-blue?style=for-the-badge)](../README.md)
[![LAB Suivant](https://img.shields.io/badge/LAB_Suivant_→-green?style=for-the-badge)](LAB2_SSH_BRUTEFORCE_DETECTION.md)

---

**Fait avec ❤️ pour la communauté SOC/Blue Team**

</div>
