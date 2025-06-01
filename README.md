# Infrastructure

#### Projet complet clé en main destiné aux étudiants en TSPCR/Administration systèmes et réseaux, intégrant Windows Server, Windows Client, Ubuntu Server, ainsi que des outils de supervision et gestion comme Zabbix, GLPI, Wireshark, le tout avec Active Directory et les services essentiels (DNS, FTP(S), Web, GPO, partage, sécurité…).

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

- Configuration IP statique : `172.28.10.10/16`, DNS primaire : `127.0.0.1`

- Renommer l’ordinateur

### 2. Installation et configuration d'Active Directory & DNS

- Promouvoir le serveur en contrôleur de domaine : `domaine.local`

- Créer une OU `ENTREPRISE`

- Créer des sous-OU : `Utilisateurs`, `Groupes`, `Ordinateurs`, `Serveurs`

- Ajouter un utilisateur `etudiant1` et un groupe `Informatique`

### 3. Installation du rôle IIS (Web) HTTPS

- Installer le rôle **IIS** (serveur web Windows)

- Créer un site web interne : `intranet.domaine.local`

- Tester avec le client

### 4. Installation et configuration de FTPS

- Installer le rôle FTP Server

- Configurer un répertoire FTP sécurisé avec certificat SSL (auto-signé possible)

- Autoriser le groupe `Informatique` uniquement

- Tester avec `FileZilla` client

### 5. Création et application de GPO

- **A - GPO** `Sécurité poste client` :

Objectif : Renforcer la sécurité des postes clients.

- **Désactiver l'accès au panneau de configuration**

  `Configuration utilisateur > Modèles d'administration > Panneau de configuration > Interdire l'accès...`

- **Forcer un mot de passe complexe**

  `Configuration ordinateur > Paramètres Windows > Paramètres de sécurité > Stratégie de mot de passe > Exiger la complexité`

- **B - GPO** `Partage réseau` :

Objectif : Automatiser le montage d’un lecteur réseau.

- **Monter automatiquement un lecteur réseau partagé**

  Exemple : `\\SRV-AD-01\Partage` mappé en Z:\

  `Configuration utilisateur > Préférences > Paramètres Windows > Lecteurs mappés`

- **C - GPO** Lier les GPO aux OU concernées:

- Dans la console **Group Policy Management**, fais un clic droit sur l’**OU concernée** (ex. `Utilisateurs` ou `Clients`) >

"**Lier un GPO existant**" > Choisis :

- `Sécurité poste client`

- `Partage réseau`

### 6. Configuration du partage de fichiers

- **Étape 1 : Créer un dossier partagé**

  - Créer le dossier : `D:\Partage`

  `Tu peux créer ce dossier sur une partition dédiée ou sur le disque principal.`

- **Étape 2 : Activer le partage réseau**

  - Clique droit sur `D:\Partage` > **Propriétés** > **Partage**

  - Cliquer sur **Partager**...

  - Ajouter les utilisateurs ou groupes (ex. `Informatique`, `Etudiant1`)

  - Définir les **autorisations de partage** (lecture/écriture)

- **Étape 3** : Appliquer les autorisations **NTFS**

  - Onglet **Sécurité** (toujours dans les propriétés du dossier)

  - Cliquer sur **Modifier**... pour ajouter des utilisateurs/groupes

  - Définir les **droits NTFS** (plus puissants et prioritaires que les droits de partage) :

```sh
| Utilisateur / Groupe | Autorisation NTFS  |
| -------------------- | ------------------ |
| `Informatique`       | **Contrôle total** |
| `Etudiant1`          | **Lecture seule**  |
```

##### Rappel important :

- Les **droits effectif**s = les plus restrictifs entre **partage** et **NTFS**.

- Exemple : Si un utilisateur a **écriture** en NTFS mais seulement **lecture** dans le partage → il n’aura que **lecture**.

### 7. Politique de sécurité (GPO)

- **GPO : Verrouillage de session après 10 minutes**

- GPO : `Verrouillage automatique`

- Chemin : `Configuration utilisateur > Modèles d'administration > Panneau de configuration > Personnalisation`

  - Activer l’écran de veille

  - Temps d’inactivité : **600 secondes (10 min)**

  - Mot de passe à la reprise : **Activé**

- **GPO : Restriction des ports (pare-feu Windows)**

- GPO : Pare-feu sécurité

- Chemin : `Configuration ordinateur > Paramètres Windows > Paramètres de sécurité > Pare-feu Windows avec fonctions avancées de sécurité`

  - Ajouter des règles de **trafic entrant** :

    - Autoriser ou bloquer des ports (ex. bloquer port 21 FTP, autoriser 3389 RDP)

- **GPO : Configuration d’audit de connexion**
- GPO : Audit sécurité

- Chemin : `Configuration ordinateur > Paramètres Windows > Paramètres de sécurité > Stratégies locales > Stratégie d’audit`

  - Activer :

    - **Audit des connexions aux comptes**

    - **Audit des événements de connexion**

  - Coche **Succès et Échecs**

- Consulter les journaux dans l’**Observateur d’événements** : Journal Windows > Sécurité

  - `ID 4624 = Connexion réussie`

  - `ID 4625 = Connexion échouée`

## Installation du Windows Client

### 8. Configuration de `CLIENT-01`

- IP statique

- Joindre le domaine `domaine.local`

- Se connecter avec `etudiant1`

- Vérifier les GPO appliquées

- Tester le partage et l’accès FTP/Web

## Serveur Ubuntu pour Zabbix

### 9. Installation de Zabbix Server sur Ubuntu

- Ubuntu Server : `172.28.10.20`

- Installer MariaDB, Apache, PHP, Zabbix Server + frontend

- Ajouter le serveur Windows à la supervision (agent Zabbix pour Windows)

### 10. Zabbix Agent sur Windows et Linux

- Installer agent sur `SRV-AD-01`

- Configurer agent pour pointer vers `172.28.10.20`

- Ajouter hôtes dans Zabbix UI

- Surveiller RAM, CPU, espace disque

- Configurer Alertes de notifications utilisant Gmail

## Installation de GLPI avec LDAP

### 11. GLPI sur Ubuntu et intégré à Active Directory

- Installer GLPI (Apache, MariaDB, PHP)

- Activer et configurer le **plugin LDAP**

- Connexion à `domaine.local`

- Importer les utilisateurs du domaine

- Ajouter `fusioninventory` pour l’inventaire automatique

- Tester une connexion avec un compte Active Directory

## Analyse avec Wireshark

### 12. Capture réseau

- Filtrer le trafic DNS, FTP, HTTP, LDAP

- Analyser les paquets de connexion au domaine, FTP, Web

- Exemple de filtres :

  - `ip.addr == 172.28.10.10`

  - `tcp.port == 389` (LDAP)

  - `ftp` ou `http`
