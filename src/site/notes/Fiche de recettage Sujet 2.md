---
{"dg-publish":true,"permalink":"/fiche-de-recettage-sujet-2/","dg-note-properties":{}}
---

# CCF E6-BTS SIO : Fiche de recettage - Sujet 2

**Sécurisation d'une infrastructure DMZ** Stormshield SNS EVA1 | Apache | SSH | Filtrage | NAT | DNS | Netcat

Ce document permet de vérifier que votre matériel et vos compétences sont prêts pour l'épreuve. Les paramètres réseau et la matrice de flux seront différents le jour J.

---

### 1. Validation des machines virtuelles

### 1.1 VM Stormshield SNS EVA1

| **Vérification**                                      | **Commande / Méthode** |
| ----------------------------------------------------- | ---------------------- |
| La VM démarre sans erreur sous VirtualBox             | Paramètres VM          |
| 3 interfaces réseau sont configurées dans VirtualBox  | Paramètres VM          |
| L'interface web du SNS est accessible en config usine | -                      |
| Je connais les identifiants par défaut du SNS         | -                      |
| Je sais remettre le SNS en configuration usine        | `defaultconfig`        |


### 1.2 VM Serveur GLPI (ou simple page web)

| **Vérification**                             | **Commande / Méthode**     |
| -------------------------------------------- | -------------------------- |
| La VM démarre sans erreur                    | -                          |
| Apache est installé et fonctionne            | `systemctl status apache2` |
| GLPI est accessible en HTTP sur le port 80   | `curl http://localhost`    |
| Le serveur SSH est installé et actif         | `systemctl status sshd`    |
| Outils réseau installés (curl, nc, ss, nmap) | `which curl nc ss nmap`    |
| Python 3 est installé                        | `python3 --version`        |
| Docker est installé (pour Technitium DNS)    | `docker --version`         |
| L'image Technitium est téléchargée           | `docker images             |

### 1.3 VM Poste client

| **Vérification**                       | **Commande / Méthode** |
| -------------------------------------- | ---------------------- |
| La VM démarre avec interface graphique | -                      |
| Un navigateur web est installé         | -                      |
| Changer les paramètres IP d'une VM     | `nmtui`                |
| curl est installé                      | `curl --version`       |
| netcat (nc) est installé               | `nc -h`                |
| nmap est installé                      | `nmap --version`       |
| ssh client est installé                | `ssh -V`               |
| nslookup ou dig est installé           | `which nslookup`       |

---

### 2. Validation des compétences techniques

### 2.1 Configuration réseau Linux

| **Test à réaliser**                     | **Commande**       |
| --------------------------------------- | ------------------ |
| Changer les paramètres IP d'une VM      | `nmtui`            |
| Vérifier la connectivité entre deux VMs | `ping IP_autre_VM` |

### 2.2 Changement de port Apache

|**Test à réaliser**|**Commande**|**Résultat attendu**|
|---|---|---|
|Modifier le port dans `ports.conf`|`nano /etc/apache2/ports.conf`|`Listen NOUVEAU_PORT`|
|Modifier le VirtualHost|`nano /etc/apache2/sites-enabled/*.conf`|`<VirtualHost *:PORT>`|
|Redémarrer Apache|`systemctl restart apache2`|Service actif|
|Vérifier l'écoute du port|`ss -tinp|grep apache`|
|Accéder à GLPI (nouveau port)|`curl http://localhost:PORT`|Page GLPI affichée|
|Vérifier l'ancien port (80)|`curl http://localhost:80`|Connexion refusée|

### 2.3 Changement de port SSH

|**Test à réaliser**|**Commande**|**Résultat attendu**|
|---|---|---|
|Modifier le port dans `sshd_config`|`nano /etc/ssh/sshd_config`|`Port NOUVEAU_PORT`|
|Redémarrer le service SSH|`systemctl restart sshd`|Service actif|
|Vérifier l'écoute du port|`ss -tinp|grep sshd`|
|Connexion SSH (nouveau port)|`ssh -p PORT user@IP`|Connexion réussie|
|Vérifier l'ancien port (22)|`ssh -p 22 user@IP`|Connexion refusée|

### 2.4 Utilisation de netcat (simulation de services) (suite)

| **Test à réaliser**                                  | **Commande**                  | **Résultat attendu** |
| ---------------------------------------------------- | ----------------------------- | -------------------- |
| Lancer un serveur TCP sur un port arbitraire         | `nc -l -p 9999`               | En écoute            |
| Se connecter au serveur TCP depuis une autre machine | `nc -zv IP 9999`              | Connection succeeded |
| Lancer un serveur UDP                                | `nc -u -l -p 514`             | En écoute UDP        |
| Envoyer un message UDP                               | `echo test                    | nc -u IP 514`        |
| Lancer un serveur HTTP rapide avec Python            | `python3 -m http.server 8443` | Serving on port 8443 |

### 2.5 Stormshield SNS (Filtrage et NAT)

| **Test à réaliser**                          | **Commande / Méthode**           | **Résultat attendu**  |
| -------------------------------------------- | -------------------------------- | --------------------- |
| Créer une règle de filtrage (Passer/Bloquer) | Politique de sécurité > Filtrage | Règle active          |
| Créer une règle NAT/PAT (SNAT ou DNAT)       | Politique de sécurité > NAT      | Règle NAT active      |
| Consulter les logs de filtrage en temps réel | Monitoring > Logs                | Logs visibles         |
| Identifier un flux bloqué dans les logs      | Filtrer les logs sur "bloqué"    | Flux bloqué identifié |

### 2.6 DNS Technitium

| **Test à réaliser**            | **Commande**                  | **Résultat attendu** |
| ------------------------------ | ----------------------------- | -------------------- |
| Lancer le conteneur Technitium | `docker compose up -d`        | Conteneur actif      |
| Accéder à l'interface web      | `http://localhost:5380`       | Interface visible    |
| Créer une zone                 | Interface web > Zones         | Zone créée           |
| Créer un enregistrement A      | Interface web                 | Enregistrement créé  |
| Tester la résolution DNS       | `nslookup nom.domaine IP_DNS` | Résolution OK        |
| Arrêter le conteneur           | `docker compose down`         | Conteneur supprimé   |
