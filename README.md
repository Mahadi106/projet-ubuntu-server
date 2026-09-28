 # Mise en place et administration d'un serveur Linux (Ubuntu Server)

## 🎯 Objectif du projet

Déployer et administrer un serveur Linux complet à partir de zéro : installation, configuration réseau, gestion des utilisateurs et des permissions, sécurisation via SSH et pare-feu, hébergement d'un site web avec Apache, et exercices de dépannage. Ce projet couvre les bases essentielles de l'administration système Linux.

## Environnement

| Élément | Détail |
|---|---|
| Hyperviseur | VMware Workstation |
| Système invité | Ubuntu Server 26.04.1 LTS (64-bit) |
| RAM allouée | 6 Go |
| Disque | 25 Go (fichier unique) |
| Mode réseau | Bridged (pont réseau) |
| Nom de la machine | `ubuntu-server` |
| Utilisateur admin | `admin235` |

Le mode **Bridged** permet à la VM d'obtenir une adresse IP visible sur le réseau local, comme un appareil physique — condition nécessaire pour l'accès SSH depuis une autre machine.

## Configuration réseau

Une adresse IP fixe a été configurée via **Netplan** (`/etc/netplan/00-installer-config.yaml`), à la place du DHCP par défaut :

```yaml
network:
  ethernets:
    ens33:
      dhcp4: false
      dhcp6: false
      addresses:
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.254
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
      set-name: ens33
  version: 2
```

Application : `sudo netplan apply` → adresse IP fixe confirmée via `ip a` / `ip route` (route marquée `proto static`).

##  Gestion des utilisateurs et permissions

- Création de deux utilisateurs : `alice` et `bob` (`sudo adduser`)
- Création d'un groupe partagé `ESGIS` (`sudo groupadd`) et ajout des deux utilisateurs (`sudo usermod -aG ESGIS <user>`)
- Création d'un dossier partagé `/home/dossier_partager` :
  - `sudo chown admin235:ESGIS /home/dossier_partager`
  - `sudo chmod 770 /home/dossier_partager` (accès complet au propriétaire et au groupe uniquement)
- **Test validé** : `alice` crée un fichier dans le dossier partagé, `bob` (même groupe) le lit avec succès.

## SSH et UFW

- **SSH** : serveur OpenSSH installé dès l'installation du système. Connexion à distance testée avec succès depuis un terminal Windows :
  ```
  ssh admin235@192.168.1.100
  ```
- **UFW (pare-feu)** : seules les connexions nécessaires sont autorisées, activées **avant** l'enable du pare-feu pour ne pas couper l'accès SSH :
  ```
  sudo ufw allow OpenSSH
  sudo ufw allow 'Apache'
  sudo ufw enable
  ```
- Statut final : `OpenSSH` et `Apache` en `ALLOW`, tout le reste bloqué par défaut.

##  Installation d'Apache

```
sudo apt install apache2 -y
```

- Service vérifié actif via `sudo systemctl status apache2`
- Page par défaut testée depuis le navigateur (`http://192.168.1.100`)
- Page personnalisée déployée dans `/var/www/html/index.html` (HTML simple avec identité du projet)

## Tests et résolution de problèmes

Trois pannes ont été simulées volontairement pour illustrer la démarche **symptôme → diagnostic → correction → vérification** :

1. **Service Apache arrêté** (`systemctl stop apache2`) → diagnostiqué via `systemctl status` → résolu avec `systemctl start apache2`
2. **Permissions cassées sur `index.html`** (`chmod 000`) → erreur 403 → corrigé avec `chmod 644`
3. **Conflit de règles UFW** (`ufw deny http` ajoutée après une règle `allow`) → observation que l'ordre des règles UFW est déterminant (la première règle correspondante l'emporte) → nettoyage avec `ufw delete deny http`

## Rapport complet

Le rapport détaillé est disponible ici :  https://github.com/Mahadi106/projet-ubuntu-server/blob/main/Rapport_Projet_Ubuntu_Server.pdf
