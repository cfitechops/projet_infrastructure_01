# Infrastructure

#### projet complet clé en main destiné aux étudiants en Technicien/Administration systèmes et réseaux, intégrant Windows Server, Windows Client, Ubuntu Server, ainsi que des outils de supervision et gestion comme Zabbix, GLPI, Wireshark, le tout avec Active Directory et les services essentiels (DNS, FTP(S), Web, GPO, partage, sécurité…).

## Objectifs pédagogiques

- Mettre en place une infrastructure d’entreprise simple mais complète.

- Comprendre le rôle de chaque service dans un réseau professionnel.

- Apprendre à configurer, sécuriser, surveiller et gérer un environnement hybride (Windows + Linux).

- Gérer utilisateurs, stratégies, autorisations, et la supervision de l'infrastructure.

## Architecture cible

```sh
+-------------------+       +---------------------+
|  Windows Server   |<----->| Ubuntu (Zabbix, GLPI)|
|  2019/2022        |       +---------------------+
| - AD DS           |                   |
| - DNS             |                   |
| - Web (IIS)       |                   |
| - FTP(S)          |                   |
| - GPO             |                   |
| - File Share      |                   |
+--------+----------+                   |
         |                              |
+--------v----------+                  |
|  Windows Client    |<----------------+
|  (Win10/11 Pro)    |
+-------------------+
```

## Machines virtuelles recommandées

```sh
| Nom de la VM     | OS                  | Rôle                                  |
| ---------------- | ------------------- | ------------------------------------- |
| `SRV-AD-01`      | Windows Server 2022 | AD DS, DNS, GPO, partage, IIS, FTP(S) |
| `CLIENT-01`      | Windows 10/11 Pro   | Client Windows du domaine             |
| `SRV-MONITOR-01` | Ubuntu Server 24.04 | Zabbix server, Zabbix agent           |
| `SRV-GLPI-01`    | Ubuntu Server 22.04 | GLPI + fusioninventory + LDAP         |
| `Wireshark`      | Kali Linux ou Win10 | Analyseur réseau                      |
```

## Étapes du projet

### 1. Installation de Windows Server

- Nom : `SRV-AD-01`

- Configuration IP statique : `192.168.10.10/24`, DNS primaire : `127.0.0.1`

- Renommer l’ordinateur

### 2. Installation et configuration d'Active Directory & DNS

- Promouvoir le serveur en contrôleur de domaine : `domaine.local`

- Créer une OU `ENTREPRISE`

- Créer des sous-OU : `Utilisateurs`, `Groupes`, `Ordinateurs`, `Serveurs`

- Ajouter un utilisateur `etudiant1` et un groupe `Informatique`

### 3. Installation du rôle IIS (Web)

- Installer le rôle **IIS**

- Créer un site web interne : `intranet.domaine.local`

- Tester avec le client

### 4. Installation et configuration de FTPS

- Installer le rôle FTP Server

- Configurer un répertoire FTP sécurisé avec certificat SSL (auto-signé possible)

- Autoriser le groupe `Informatique` uniquement

### 5. Création et application de GPO

- GPO `Sécurité poste client` :

  - Désactiver panneau de configuration

  - Forcer mot de passe complexe

- GPO `Partage réseau` :

  - Monter un lecteur réseau automatiquement (ex : `\\SRV-AD-01\Partage`)

- Lier les GPO aux OU concernées

### 6. Configuration du partage de fichiers

- Créer un dossier partagé :` D:\Partage`

- Activer **le partage réseau**

- Appliquer des autorisations **NTFS** (droits lecture/écriture)

  - `Informatique` → contrôle total

  - `Etudiant1` → lecture seule

### 7. Politique de sécurité

- GPO de verrouillage de session après 10 min

- GPO de restriction des ports (via pare-feu Windows)

- Configuration d’audit de connexion

## Installation du Windows Client

### 8. Configuration de `CLIENT-01`

- IP statique

- Joindre le domaine `domaine.local`

- Se connecter avec `etudiant1`

- Vérifier les GPO appliquées

- Tester le partage et l’accès FTP/Web

## Serveur Ubuntu pour Zabbix

### 9. Installation de Zabbix Server sur Ubuntu

- Ubuntu Server : `192.168.10.20`

- Installer MariaDB, Apache, PHP, Zabbix Server + frontend

- Ajouter le serveur Windows à la supervision (agent Zabbix pour Windows)

### 10. Zabbix Agent sur Windows et Linux

- Installer agent sur `SRV-AD-01`

- Configurer agent pour pointer vers `192.168.10.20`

- Ajouter hôtes dans Zabbix UI

- Surveiller RAM, CPU, espace disque

## Installation de GLPI avec LDAP

### 11. GLPI sur Ubuntu

- Installer Apache, MariaDB, PHP, GLPI

- Activer et configurer le **plugin LDAP**

- Connexion à `domaine.local`

- Importer les utilisateurs du domaine

- Ajouter `fusioninventory` pour l’inventaire automatique

## Analyse avec Wireshark

### 12. Capture réseau

- Filtrer le trafic DNS, FTP, HTTP, LDAP

- Analyser les paquets de connexion au domaine, FTP, Web

- Exemple de filtres :

  - `ip.addr == 192.168.10.10`

  - `tcp.port == 389` (LDAP)

  - `ftp` ou `http`
