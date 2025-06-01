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

- Nom : SRV-AD-01

- Configuration IP statique : 192.168.10.10/24, DNS primaire : 127.0.0.1

- Renommer l’ordinateur
