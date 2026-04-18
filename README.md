# homelab-wazuh
Déploiement et administration d'une solution XDR open source Wazuh en environnement de lab personnel. Documentation des étapes d'installation, configuration des agents et analyse des alertes de sécurité.

# Homelab SOC - Déploiement Wazuh XDR

## Objectif
Déploiement d'une solution XDR open source dans un environnement de lab personnel (2 W11 PRO)

## Architecture
- Serveur Wazuh Manager : [Ubuntu Server 22.04 Oracle VB]
- Agents déployés sur : [Windows 11]

## Installation
### Prérequis
- Oracle VB
- Ubuntu Server 22.04 (Wazuh fonctionne mal avec les autres versions) => Config :
- RAM : minimum 4Go recommandé
- CPU : 2 cœurs minimum
- Stockage : 50Go minimum
- 2 Vm W11 avec le réseau configuré en bridge
- Nom du réseau interne : lab-network (en cas de test avec des ransomware)
- plage ip : 192.168.100.1 -> 192.168.100.253 (la 254 étant ma passerelle par défaut pour de futurs projets)

### Étapes
1) Lancement et installation de la VM Server
2) Configuration réseau => Mode bridge pour l'installation
3) Mise à jour de la VM => sudo apt update & sudo apt upgrade
4) Installation de curl => sudo apt install curl
5) Installer Wazuh => curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
6) UFW inactif par défaut sur Ubuntu Server 22.04 — aucune règle firewall nécessaire pour le lab
7) Connexion à la console <img width="1685" height="1197" alt="image" src="https://github.com/user-attachments/assets/d488591a-984c-4655-98b5-1b6e72f49fd2" />
8) Ajout d'agents à surveiller => télécharger l'agent en version 7.4, lancer le programme d'installation sur le client, entrer l'ip du serveur Ubuntu, cliquer sur "Save"
9) Ouvrir l'invit de commande en mode admin sur le client, et entrer la commande NET START WazuhSvc
10) Le service Wazuh va démarrer
11) Ensuite, cliquer sur "Generate Key", l'agent va générer une clé et remonter automatiquement dans la console.
12) Premier agent Wazuh actif screen <img width="1678" height="1003" alt="image" src="https://github.com/user-attachments/assets/4a021f86-d22a-4bec-b621-ba8a919a2f47" />
13) Deuxième agent Wazuh actif screen <img width="1677" height="1331" alt="image" src="https://github.com/user-attachments/assets/6be60a2c-5530-42f4-80b9-8ca4f14b808b" />


### Problèmes rencontrés et solutions
1) Erreur rencontrée : No such file or directory - Cause : Oubli du "-" devant "sO". La commande correcte est donc curl -sO https:// [...]
2) Ne pas oublier de noter l'adresse ip du serveur => 192.168.1.19/24 (à utiliser pour la configuration de l'agent Wazuh)
3) Erreur lors de l'accès à l'ip du serveur Wazuh => http://192.168.1.19. Solution, il faut mettre https://192.168.1.19
4) Oubli de noter le mdp à la fin de l'install => Commande sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt pour le retrouver
5) L'agent ne récupère pas sa clé d'authentification - Cause : Mauvaise version de l'agent pour du Wazuh 4.7 - Solution : Télécharger l'agent en version 4.7.5
6) Installation d'une nouvelle VM W11 => Erreur : No bootable option or device was found — Cause : ordre de boot incorrect dans VirtualBox. Solution : Paramètres VM → Système → Carte mère → mettre Optique en premier dans l'ordre d'amorçage
7) Windows 11 refuse de s'installer sur VirtualBox sans TPM 2.0 et Secure Boot. Solution : bypass via regedit pendant l'installation — créer la clé HKEY_LOCAL_MACHINE\SYSTEM\Setup\LabConfig avec les valeurs BypassTPMCheck et BypassSecureBootCheck à 1
8) Les alertes ne remontent pas correctement dans la console Wazuh => Activation des audits avec la commande powershell : auditpol /set /category:* /success:enable /failure:enable

## Configuration
### Réseau
- Mode réseau Wazuh Manager : Bridge (192.168.1.19/24)
- Mode réseau agents (phase normale) : Bridge
- Mode réseau agents (phase malware) : Réseau Interne "lab-network"
- Plage IP réseau interne : 192.168.100.1 → 192.168.100.253
- Passerelle réservée : 192.168.100.254 (futurs projets)

### Agents configurés
| Nom machine | OS | IP | Statut |
|-------------|----|----|--------|
| [Windows 11 Famille] | Windows 11 | 192.168.1.17 | Actif |
| [W11 Pro Lab] | Windows 11 | 192.168.1.179 | Actif |

### Versions
- Wazuh Manager : 4.7.0
- Wazuh Agent : 4.7.5
- Ubuntu Server : 22.04 LTS
## Résultats et alertes observées
### Test 1 — Simulation de création de compte administrateur suspect

Commandes lancées depuis un CMD admin sur l'agent Windows :
- net user hacker /add
- net localgroup Administrateurs hacker /add  
- net user hacker /delete

Alertes générées automatiquement par Wazuh :

| Technique MITRE | Tactique | Description | Niveau |
|-----------------|----------|-------------|--------|
| T1098 | Persistence | User account created | 8 |
| T1484 | Defense Evasion, Privilege Escalation | Administrators group changed | 12 |
| T1484 | Defense Evasion, Privilege Escalation | Users group changed | 5 |
| T1078 | Persistence, Initial Access | Failed privileged operation | 4 |
<img width="1657" height="641" alt="Alertes" src="https://github.com/user-attachments/assets/0f6d643b-90d2-4411-9f05-5a0e2d8e9956" />

## Prochaines étapes
- Intégrer pfSense comme firewall virtuel
- Monter un Active Directory
