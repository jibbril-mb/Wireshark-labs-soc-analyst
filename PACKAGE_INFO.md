# 📦 Collection Complète de LABs Wireshark pour SOC Analyste

## ✅ Contenu du Package

Vous avez maintenant accès à une collection complète de 8 fichiers PCAP réalistes simulant les incidents de sécurité les plus courants en environnement SOC.

### 📁 Fichiers Inclus

```
wireshark_labs/
├── 📄 README.md                      (Guide de démarrage rapide)
├── 📘 GUIDE_ANALYSE_LABS.md          (Instructions détaillées 50+ pages)
├── 🔧 analyze_lab.sh                 (Scripts d'analyse automatique)
│
├── 🎯 LAB1_Port_Scan.pcap           (138 KB - 2006 paquets)
├── 🎯 LAB2_SSH_BruteForce.pcap      (482 KB - 5103 paquets)
├── 🎯 LAB3_DNS_Tunneling.pcap       (5.4 KB - 48 paquets)
├── 🎯 LAB4_C2_Beacon.pcap           (28 KB - 300 paquets)
├── 🎯 LAB5_Lateral_Movement.pcap    (13 KB - 76 paquets)
├── 🎯 LAB6_Malware_Download.pcap    (47 KB - 50 paquets)
├── 🎯 LAB7_SQL_Injection.pcap       (6.1 KB - 58 paquets)
└── 🎯 LAB8_ARP_Spoofing.pcap        (2.6 KB - 40 paquets)
```

**Taille totale du package : ~722 KB**

---

## 🚀 Démarrage Rapide en 3 Étapes

### Étape 1 : Télécharger Wireshark
- **Windows** : https://www.wireshark.org/download.html
- **macOS** : `brew install --cask wireshark`
- **Linux** : `sudo apt install wireshark`

### Étape 2 : Ouvrir un LAB
1. Lancer Wireshark
2. File → Open → Sélectionner un fichier `.pcap`
3. Commencer l'analyse

### Étape 3 : Suivre le Guide
Ouvrez `GUIDE_ANALYSE_LABS.md` pour des instructions étape par étape.

---

## 🎓 Parcours d'Apprentissage Recommandé

### Niveau Débutant (Commencez ici)
1. **LAB 1** - Port Scan (⏱️ 10-15 min)
   - Concepts : Reconnaissance, TCP flags, statistiques
   - Difficulté : ⭐ Facile

2. **LAB 6** - Malware Download (⏱️ 15-20 min)
   - Concepts : HTTP, extraction de fichiers, IOCs
   - Difficulté : ⭐⭐ Moyen

### Niveau Intermédiaire
3. **LAB 2** - SSH Brute Force (⏱️ 15-20 min)
   - Concepts : Détection de patterns, compromission
   - Difficulté : ⭐⭐ Moyen

4. **LAB 4** - C2 Beacon (⏱️ 15-20 min)
   - Concepts : Analyse temporelle, malware C2
   - Difficulté : ⭐⭐ Moyen

5. **LAB 7** - SQL Injection (⏱️ 15-20 min)
   - Concepts : Attaques web, exploitation applicative
   - Difficulté : ⭐⭐ Moyen

### Niveau Avancé (Challenge)
6. **LAB 3** - DNS Tunneling (⏱️ 20-30 min)
   - Concepts : Exfiltration, encodage, détection avancée
   - Difficulté : ⭐⭐⭐ Difficile

7. **LAB 5** - Lateral Movement (⏱️ 20-25 min)
   - Concepts : SMB, PsExec, propagation réseau
   - Difficulté : ⭐⭐⭐ Difficile

8. **LAB 8** - ARP Spoofing (⏱️ 20-30 min)
   - Concepts : MITM, protocoles niveau 2, interception
   - Difficulté : ⭐⭐⭐ Difficile

**Temps total estimé : 3-4 heures pour compléter tous les LABs**

---

## 🔍 Filtres Wireshark Essentiels à Connaître

### Détection de Menaces
```
tcp.flags.syn == 1                    # Scans de ports
tcp.port == 22 && tcp.analysis.retransmission  # SSH suspect
dns && (frame.len > 100)              # DNS tunneling
http.request.uri contains "SELECT"    # SQL Injection
arp.duplicate-address-detected        # ARP Spoofing
tcp.analysis.flags && !tcp.analysis.window_update  # Anomalies TCP
```

### Analyse Générale
```
ip.addr == [IP]                       # Trafic d'une IP spécifique
tcp.port == [PORT]                    # Trafic sur un port
http.response.code >= 400             # Erreurs HTTP
dns.qry.name contains "[DOMAIN]"      # Requêtes DNS spécifiques
tcp.stream eq [NUM]                   # Suivre une conversation
frame.time >= "2024-05-20 14:00:00"  # Filtrer par temps
```

### Extraction de Données
```
http.request.method == "POST"         # Requêtes POST (souvent sensibles)
http.file_data                        # Données HTTP
tls.handshake.extensions_server_name  # SNI (domaines HTTPS)
smb2.filename                         # Fichiers SMB
```

---

## 🎯 Objectifs Pédagogiques par LAB

| LAB | Compétences Développées | IOCs à Identifier |
|-----|-------------------------|-------------------|
| **1** | Détection de reconnaissance, analyse TCP | IP attaquant, ports ouverts, type de scan |
| **2** | Détection brute force, confirmation compromission | Nombre tentatives, connexion réussie, timeline |
| **3** | Exfiltration de données, décodage | Domaine C2, données exfiltrées, volume |
| **4** | Détection malware, analyse périodique | Serveur C2, intervalle beacon, protocole |
| **5** | Mouvement latéral, détection outils | Machines compromises, outils (PsExec), timeline |
| **6** | Analyse infection, extraction artefacts | Malware téléchargé, source, callback C2 |
| **7** | Exploitation web, détection injection | Payloads SQL, données exposées, outil |
| **8** | Attaque MITM, analyse couche 2 | Attaquant, victime, données interceptées |

---

## 🛠️ Scripts d'Analyse Automatique

### Utilisation
```bash
# Rendre le script exécutable (Linux/macOS)
chmod +x analyze_lab.sh

# Analyser un LAB spécifique
./analyze_lab.sh 1    # LAB 1 - Port Scan
./analyze_lab.sh 2    # LAB 2 - SSH Brute Force
./analyze_lab.sh 3    # LAB 3 - DNS Tunneling
# etc.
```

### Prérequis
- **tshark** doit être installé (version CLI de Wireshark)
- Linux/macOS recommandé pour les scripts shell
- Windows : Utiliser WSL (Windows Subsystem for Linux)

### Ce que font les scripts
- Analyse automatique du trafic
- Extraction des IOCs
- Statistiques et métriques
- Identification des anomalies
- Génération de rapports

**💡 Astuce** : Utilisez les scripts APRÈS avoir tenté l'analyse manuelle pour vérifier vos résultats.

---

## 📊 Indicateurs de Succès

### Vous maîtrisez un LAB quand vous pouvez :
- ✅ Identifier le type d'attaque en < 2 minutes
- ✅ Extraire tous les IOCs clés
- ✅ Expliquer le déroulement de l'attaque
- ✅ Proposer des contre-mesures appropriées
- ✅ Rédiger un rapport d'incident professionnel

### Checklist de Progression
```
□ LAB 1 : Ports ouverts identifiés (4 ports)
□ LAB 2 : Compromission SSH détectée (connexion longue)
□ LAB 3 : Message secret décodé (base64)
□ LAB 4 : Intervalle beacon calculé (60 secondes)
□ LAB 5 : Timeline de propagation établie (4 machines)
□ LAB 6 : Malware extrait et hash calculé
□ LAB 7 : Données exposées identifiées (admin credentials)
□ LAB 8 : Attaquant identifié (MAC address)
```

---

## 🏆 Challenges Avancés

### Challenge 1 : Speed Run
**Objectif** : Analyser LAB 1 en moins de 5 minutes.
**Critères** : Identifier IP attaquant, type de scan, ports ouverts.

### Challenge 2 : Needle in the Haystack
**Objectif** : Trouver la connexion SSH réussie dans LAB 2.
**Indice** : C'est la connexion #458 sur 500.

### Challenge 3 : Decoder Master
**Objectif** : Décoder INTÉGRALEMENT le message dans LAB 3.
**Question** : Combien de caractères contient le message original ?

### Challenge 4 : Timeline Forensics
**Objectif** : Créer une timeline précise de LAB 5.
**Format** : HH:MM:SS → Machine cible

### Challenge 5 : Full Investigation
**Objectif** : Analyser LAB 6 comme un incident réel.
**Livrable** : Rapport complet avec recommandations.

---

## 📚 Ressources Complémentaires

### Documentation Officielle
- **Wireshark User Guide** : https://www.wireshark.org/docs/
- **Display Filter Reference** : https://www.wireshark.org/docs/dfref/
- **Wiki Wireshark** : https://wiki.wireshark.org/

### Sites de Pratique (PCAPs Réels)
- **Malware-Traffic-Analysis.net** ⭐ Excellent !
- **NetreSec.com** - PCAP Gallery
- **CyberDefenders.org** - Blue Team Labs
- **TryHackMe** - Wireshark Rooms

### Chaînes YouTube Recommandées
- **Chris Greer** - Tutorials Wireshark (pédagogue excellent)
- **13Cubed** - DFIR & Network Forensics
- **David Bombal** - Networking & Security

### Outils Complémentaires
- **NetworkMiner** - Extraction automatique d'artefacts
- **Zeek (Bro)** - Analyse réseau avec scripts
- **Suricata** - IDS/IPS open source
- **CyberChef** - Encodage/décodage (base64, hex, etc.)

---

## 🎓 Certifications Associées

Après avoir maîtrisé ces LABs, vous serez prêt pour :

### Entry Level
- **CompTIA Security+** (SY0-701)
- **CompTIA CySA+** (CS0-003)

### Intermediate
- **Wireshark Certified Network Analyst (WCNA)**
- **Blue Team Level 1 (BTL1)** - Security Blue Team

### Advanced
- **GIAC Certified Intrusion Analyst (GCIA)**
- **GIAC Network Forensic Analyst (GNFA)**

### Roles Visés
- **SOC Analyst Level 1/2**
- **Incident Responder**
- **Security Analyst**
- **Network Security Engineer**
- **Threat Hunter**

---

## ❓ FAQ

### Q1 : Puis-je utiliser ces LABs pour ma formation professionnelle ?
**R** : Oui ! Ces fichiers sont créés à des fins éducatives et utilisables librement pour la formation.

### Q2 : Les données sont-elles réelles ?
**R** : Non. Tous les fichiers PCAP sont des simulations pédagogiques générées avec Scapy. Aucune donnée réelle ou sensible.

### Q3 : Quel temps faut-il pour compléter tous les LABs ?
**R** : Environ 3-4 heures pour une première analyse. Plus si vous approfondissez.

### Q4 : Faut-il des connaissances réseau avancées ?
**R** : Non. Les LABs commencent au niveau débutant. Le guide explique tous les concepts.

### Q5 : Puis-je utiliser ces fichiers en entretien d'embauche ?
**R** : Oui ! C'est un excellent moyen de démontrer vos compétences pratiques en analyse réseau.

### Q6 : Les scripts fonctionnent-ils sur Windows ?
**R** : Les scripts shell (.sh) nécessitent Linux/macOS ou WSL sous Windows. Mais l'analyse manuelle avec Wireshark fonctionne partout.

### Q7 : Comment vérifier mes réponses ?
**R** : Consultez la section "Solutions des LABs" dans GUIDE_ANALYSE_LABS.md ou utilisez les scripts automatiques.

### Q8 : Puis-je créer mes propres LABs ?
**R** : Absolument ! Les scripts Python utilisés (Scapy) sont disponibles. Vous pouvez les modifier pour créer de nouveaux scénarios.

---

## 🤝 Contribution & Feedback

### Vous avez des suggestions ?
- Nouveaux scénarios d'attaque à ajouter
- Améliorations du guide pédagogique
- Bugs dans les scripts d'analyse
- Idées de challenges

### Partager Vos Résultats
Après avoir complété les LABs, n'hésitez pas à :
- Partager vos rapports d'analyse
- Proposer des améliorations
- Créer vos propres variantes

---

## 📜 Licence & Utilisation

### Conditions d'Utilisation
✅ **Autorisé** :
- Utilisation personnelle pour apprentissage
- Formation professionnelle et académique
- Démonstrations en entretien
- Ateliers et conférences
- Modification et création de variantes

❌ **Non autorisé** :
- Revente des fichiers
- Utilisation à des fins malveillantes
- Prétendre à la paternité du contenu

### Attribution
Ces LABs ont été créés à des fins pédagogiques pour la communauté SOC/Blue Team.
Libre d'utilisation avec mention de la source si redistribué.

---

## 🎯 Prochaines Étapes

### Après avoir complété ces LABs :

1. **Pratiquer sur PCAPs réels**
   - Malware-Traffic-Analysis.net (recommandé)
   - CyberDefenders.org

2. **Approfondir les concepts**
   - Suivre des cours Wireshark avancés
   - Étudier les RFCs des protocoles

3. **Automatiser l'analyse**
   - Apprendre Zeek/Suricata
   - Créer vos propres scripts d'analyse

4. **Participer à la communauté**
   - Reddit : r/blueteamsec, r/netsec
   - Discord : Blue Team villages
   - Conferences : DEF CON Blue Team Village

5. **Préparer une certification**
   - WCNA pour Wireshark spécifiquement
   - CySA+ pour SOC en général

---

## 📞 Support

### Besoin d'Aide ?

1. **Consultez d'abord** :
   - README.md (vue d'ensemble)
   - GUIDE_ANALYSE_LABS.md (instructions détaillées)
   - Documentation Wireshark officielle

2. **Ressources communautaires** :
   - Wireshark Q&A : https://ask.wireshark.org/
   - Reddit : r/wireshark
   - Stack Exchange : Security

3. **Problèmes techniques** :
   - Vérifiez que Wireshark est à jour
   - Testez avec un autre LAB pour isoler le problème
   - Consultez les logs d'erreur

---

## 🎊 Remerciements

Ces LABs sont dédiés à tous les analystes SOC qui protègent nos systèmes chaque jour.

**"The network never lies. It just needs someone to listen."**

Bon courage dans votre apprentissage et votre carrière en cybersécurité ! 🛡️🔍

---

**Dernière mise à jour** : Mai 2024  
**Version** : 1.0  
**Package** : 8 LABs + 3 guides + Scripts automatiques  
**Taille totale** : ~722 KB  
