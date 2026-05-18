# Projet-Final-ASI
**Université :** Institut Universitaire Des Sciences (IUS)  
**Faculté :** Faculté des Sciences et des Technologies (FST)  
**Préparé par :**  
- Donsam Jean Gabard NOEL  
- Christy Gérys LAMBERT  
- Lens Sandro PETIOTE  

**Sous la Direction de :** M.Ismael SAINT-AMOUR 

**Date :** 18 mai 2026  

---

## Table des matières

1. [Introduction](#1-introduction)
2. [Objectif du projet](#2-objectif-du-projet)
3. [Problématique](#3-problématique)
4. [Méthodologie](#4-méthodologie)
5. [Description du projet](#5-description-du-projet)
6. [Tests de connexion](#6-tests-de-connexion)
7. [Récapitulatif des mesures](#7-récapitulatif-des-mesures)
8. [Conclusion](#8-conclusion)

---

## 1. Introduction

### Contexte

La sécurisation des accès distants constitue l'une des priorités fondamentales de l'administration système moderne. SSH (Secure Shell) est le protocole de référence pour la gestion à distance des serveurs Linux, mais sa configuration par défaut n'offre pas un niveau de sécurité optimal face aux menaces actuelles telles que les attaques par force brute, les tentatives d'intrusion automatisées ou l'exploitation de comptes privilégiés.

Dans le cadre de ce projet, réalisé en équipe de trois personnes, nous avons mis en place un environnement de laboratoire sur VirtualBox afin d'expérimenter concrètement le durcissement (hardening) d'un serveur OpenSSH. 

Ce rapport documente l'ensemble des étapes réalisées : l'authentification par clé cryptographique, la désactivation du compte root, le changement du port SSH, l'installation de Fail2ban, et la limitation des utilisateurs autorisés. Pour chaque étape, nous présentons la configuration appliquée ainsi que les résultats des tests de connexion effectués.


---

## 2. Objectifs du projet

#### Objectif général
L'objectif principal de ce projet est de renforcer la sécurité d'un serveur OpenSSH en appliquant les meilleures pratiques d'administration système, en simulant un environnement réel sur VirtualBox.
 
#### Objectifs spécifiques
- Mettre en place une authentification par paire de clés cryptographiques (clé publique/privée) en remplacement de l'authentification par mot de passe.
- Désactiver la connexion directe du compte root via SSH afin de réduire la surface d'attaque.
- Changer le port d'écoute SSH du port par défaut (22) vers un port personnalisé afin de limiter les attaques automatisées.
- Installer et configurer Fail2ban pour bannir automatiquement les adresses IP suspectes après plusieurs tentatives de connexion échouées.
- Restreindre la liste des utilisateurs autorisés à se connecter via SSH grâce à la directive AllowUsers.

---

## 3. Problématique
Les serveurs exposés sur Internet sont continuellement soumis à des tentatives d'intrusion automatisées. Les robots (bots) scannent en permanence le port 22, tentent de se connecter avec des identifiants par défaut ou lancent des attaques par dictionnaire sur des comptes actifs, notamment root. Une configuration SSH par défaut laisse donc la porte ouverte à de nombreuses vulnérabilités.

La problématique de ce projet peut ainsi se formuler de la manière suivante : Comment sécuriser efficacement un serveur OpenSSH en appliquant plusieurs couches de protection complémentaires ?

---

## 4. Méthodologie

### Mise en place de l'environnement
Chaque membre du groupe a installé VirtualBox sur sa machine personnelle et créé une machine virtuelle :
- Une VM Serveur sous Ubuntu Server 22.04 LTS, qui joue le rôle du serveur SSH à durcir.
- Les tests de connexion SSH ont été effectués directement depuis la machine personnelle de chaque membre, sans VM cliente supplémentaire.

La machine personnelle de chaque membre communique avec la VM Serveur via le réseau Host-Only de VirtualBox, ce qui permet de simuler un environnement isolé et sécurisé.

---

## 5. Description du projet

Cette section présente les cinq étapes techniques réalisées pour durcir le serveur OpenSSH, telles qu'elles ont été appliquées sur la machine virtuelle Ubuntu Server 22.04 LTS.

---

### Étape 1 — Authentification par clé SSH

#### Objectif

Remplacer l'authentification par mot de passe (vulnérable aux attaques par force brute) par une paire de clés cryptographiques (clé privée sur le client, clé publique sur le serveur).

#### Étapes réalisées

##### 1.1 Génération de la paire de clés (depuis la machine personnelle)

```bash
ssh-keygen -t ed25519 -C "admin@serveur-ssh"

# Résultat attendu :
# Generating public/private ed25519 key pair.
# Enter file in which to save the key (/home/user/.ssh/id_ed25519):
# Enter passphrase (empty for no passphrase): [entrez une passphrase]
```

> **À remplir :** Décrivez ici ce que vous avez observé lors de la génération. Quel chemin avez-vous choisi pour la clé ?

##### 1.2 Déploiement de la clé publique sur le serveur

```bash
# Méthode 1 : avec ssh-copy-id (recommandé)
ssh-copy-id -i ~/.ssh/id_ed25519.pub utilisateur@IP_SERVEUR

# Méthode 2 : manuellement (si ssh-copy-id non disponible)
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

> **À remplir :** Quelle méthode avez-vous utilisée ? Y a-t-il eu des erreurs ?

##### 1.3 Modification du fichier sshd_config

Fichier modifié : `/etc/ssh/sshd_config`

```
PubkeyAuthentication yes
PasswordAuthentication no
AuthorizedKeysFile .ssh/authorized_keys
```

```bash
# Redémarrage du service SSH après modification
sudo systemctl restart sshd

# Vérification que le service est bien actif
sudo systemctl status sshd
```

#### Résultat obtenu

> **À remplir :** La connexion par clé fonctionne-t-elle ? La connexion par mot de passe est-elle bien refusée ?

**Screenshot :**
![Connexion SSH par clé réussie](../screenshots/auth_cle_reussie.png)

---

### Étape 2 — Désactivation du login root

#### Objectif

Le compte `root` est la cible principale des attaques automatisées. Le désactiver force les attaquants à deviner en plus le nom d'utilisateur, réduisant ainsi considérablement la surface d'attaque.

#### Étapes réalisées

##### 2.1 Modification du fichier sshd_config

```
PermitRootLogin no
```

```bash
sudo systemctl restart sshd
```

##### 2.2 Test de vérification — connexion root (doit être refusée)

```bash
ssh root@IP_SERVEUR -p PORT_SSH

# Message attendu :
# Permission denied (publickey).
# ou : root@IP_SERVEUR: Permission denied
```

#### Résultat obtenu

> **À remplir :** La connexion root est-elle bien refusée ? Copiez ici le message d'erreur obtenu.

**Screenshot :**
![Connexion root refusée](../screenshots/root_refuse.png)

---

### Étape 3 — Changement du port SSH

#### Objectif

Le port 22 est scanné en permanence par des robots automatisés. Changer le port d'écoute réduit considérablement le nombre de tentatives d'intrusion visibles dans les journaux système.

#### Étapes réalisées

##### 3.1 Modification du fichier sshd_config

```
Port 2222
```

##### 3.2 Ouverture du nouveau port dans le pare-feu AVANT de redémarrer SSH

```bash
sudo ufw allow 2222/tcp
sudo ufw status
```

##### 3.3 Redémarrage du service SSH

```bash
sudo systemctl restart sshd
```

> **Note importante — Ubuntu récent :** Sur les versions récentes d'Ubuntu, SSH est géré par un **socket systemd** qui contrôle le port, et non uniquement par `sshd_config`. Si le changement de port n'a pas d'effet, il faut modifier le socket SSH :

```bash
sudo systemctl edit ssh.socket
```

Dans l'éditeur qui s'ouvre, coller exactement ceci :

```ini
[Socket]
ListenStream=
ListenStream=2222
```

##### 3.4 Vérification et test depuis un autre terminal

```bash
# Vérifier que SSH écoute sur le nouveau port
ss -tlnp | grep sshd

# Tester la connexion sur le nouveau port
ssh -p 2222 utilisateur@IP_SERVEUR

# Vérifier que l'ancien port 22 est bien fermé
ssh -p 22 utilisateur@IP_SERVEUR
# Message attendu : Connection refused
```

#### Résultat obtenu

> **À remplir :** Le nouveau port fonctionne-t-il ? L'ancien port 22 est-il bien inaccessible ?

**Screenshot :**
![Connexion sur le nouveau port](../screenshots/nouveau_port.png)

---

### Étape 4 — Mise en place de Fail2ban

#### Objectif

Fail2ban surveille les journaux SSH et bloque automatiquement les adresses IP qui échouent trop souvent à s'authentifier, protégeant ainsi le serveur contre les attaques par force brute.

#### Étapes réalisées

##### 4.1 Installation

```bash
sudo apt update
sudo apt install fail2ban -y
```

##### 4.2 Création du fichier de configuration local

> **Important :** Ne jamais modifier `jail.conf` directement. Créer un fichier `jail.local` qui le surcharge.

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

##### 4.3 Modification de la section [sshd]

```ini
[DEFAULT]
bantime  = 3600
findtime = 600
maxretry = 3

[sshd]
enabled  = true
port     = 2222
logpath  = /var/log/auth.log
backend  = systemd
```

##### 4.4 Activation et démarrage de Fail2ban

```bash
sudo systemctl enable fail2ban
sudo systemctl restart fail2ban
```

##### 4.5 Vérification du statut

```bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

##### 4.6 Test du bannissement

```bash
# Simuler plusieurs connexions échouées (répéter 3 fois)
ssh utilisateur_inexistant@IP_SERVEUR -p 2222

# Vérifier que l'IP est bien bannie
sudo fail2ban-client status sshd
# La section "Banned IP list" doit afficher l'IP testée
```

#### Résultat obtenu

> **À remplir :** Combien de tentatives avant le ban ? L'IP est-elle bien apparue dans la liste des adresses bannies ?

**Screenshot :**
![Fail2ban - IP bannie](../screenshots/fail2ban_ban.png)

---

### Étape 5 — Limitation des utilisateurs autorisés

#### Objectif

Restreindre les connexions SSH uniquement aux utilisateurs explicitement listés, même si d'autres comptes existent sur le serveur.

#### Étapes réalisées

##### 5.1 Modification du fichier sshd_config

```
AllowUsers adminuser
# Exemple avec plusieurs utilisateurs : AllowUsers alice bob charlie
```

```bash
sudo systemctl restart sshd
```

##### 5.2 Tests de vérification

```bash
# Test 1 : connexion avec un utilisateur AUTORISÉ → doit réussir
ssh -p 2222 adminuser@IP_SERVEUR

# Test 2 : connexion avec un utilisateur NON AUTORISÉ → doit échouer
ssh -p 2222 autreuser@IP_SERVEUR
# Message attendu : Permission denied
```

#### Résultat obtenu

> **À remplir :** Les utilisateurs non autorisés sont-ils bien bloqués ? Quel message d'erreur obtenez-vous ?

**Screenshot :**
![Utilisateur non autorisé refusé](../screenshots/allowusers_refuse.png)

---

## 7. Tests de connexion

### Récapitulatif des tests effectués

| Test | Commande utilisée | Résultat attendu | Résultat obtenu |
|------|-------------------|-----------------|-----------------|
| Connexion par clé | `ssh -p 2222 -i ~/.ssh/id_ed25519 user@IP` | Succès | _(À remplir)_ |
| Connexion par mot de passe | `ssh -p 2222 user@IP` (sans clé) | Refusé | _(À remplir)_ |
| Connexion root | `ssh -p 2222 root@IP` | Refusé | _(À remplir)_ |
| Connexion port 22 | `ssh -p 22 user@IP` | Connection refused | _(À remplir)_ |
| Connexion nouveau port | `ssh -p 2222 user@IP` | Succès | _(À remplir)_ |
| Brute-force → Fail2ban | 3 échecs consécutifs | IP bannie | _(À remplir)_ |
| Utilisateur non autorisé | `ssh -p 2222 autreuser@IP` | Refusé | _(À remplir)_ |

### Script de test automatisé

Le script `scripts/test_connexion.sh` a été utilisé pour automatiser ces tests.

```bash
# Exécution du script de test
chmod +x scripts/test_connexion.sh
./scripts/test_connexion.sh
```

> **À remplir :** Collez ici la sortie du script ou décrivez les résultats observés.

---

## 8. Récapitulatif des mesures

| Mesure de sécurité | Directive sshd_config | Valeur appliquée | Statut |
|--------------------|-----------------------|-----------------|--------|
| Authentification par clé | `PubkeyAuthentication` | `yes` | _(✅ / ❌)_ |
| Mot de passe désactivé | `PasswordAuthentication` | `no` | _(✅ / ❌)_ |
| Login root désactivé | `PermitRootLogin` | `no` | _(✅ / ❌)_ |
| Port SSH modifié | `Port` | _(votre port)_ | _(✅ / ❌)_ |
| Fail2ban actif | `jail.local` configuré | maxretry=3 | _(✅ / ❌)_ |
| Utilisateurs restreints | `AllowUsers` | _(vos users)_ | _(✅ / ❌)_ |

---

## 9. Conclusion

### Ce que nous avons réalisé

> **À remplir :** Résumez en quelques phrases les 5 mesures que vous avez appliquées et leur effet sur la sécurité du serveur.

### Difficultés rencontrées

> **À remplir :** Décrivez les problèmes rencontrés pendant le projet. Par exemple :
> - Erreur lors du redémarrage de SSH après modification du port
> - Fail2ban ne détectait pas les logs au bon endroit
> - Clé SSH non reconnue (problème de permissions sur authorized_keys)

### Ce que nous avons appris

> **À remplir :** Qu'est-ce que ce projet vous a appris sur l'administration système et la sécurité SSH ?

### Perspectives d'amélioration

Pour aller plus loin dans le durcissement du serveur, on pourrait envisager :

- **Authentification à deux facteurs (2FA)** avec Google Authenticator + PAM
- **Port knocking** : le port SSH ne s'ouvre que sur une séquence de paquets spécifique
- **Restriction par adresse IP** avec `AllowUsers user@192.168.1.*`
- **Désactivation des algorithmes de chiffrement faibles** dans sshd_config
- **Audit régulier** des logs avec `journalctl -u ssh`

---

## Annexes

### Fichier sshd_config complet (après durcissement)

```
# /etc/ssh/sshd_config – Configuration durcie
# Généré dans le cadre du Projet 6 – FST

Port 2222                        # Remplacez par votre port
AddressFamily inet
ListenAddress 0.0.0.0

# Authentification
PermitRootLogin no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
PasswordAuthentication no
PermitEmptyPasswords no
ChallengeResponseAuthentication no

# Restriction des utilisateurs
AllowUsers adminuser             # Remplacez par vos utilisateurs

# Timeout et sessions
LoginGraceTime 30
MaxAuthTries 3
MaxSessions 5
ClientAliveInterval 300
ClientAliveCountMax 2

# Désactiver les fonctions non utilisées
X11Forwarding no
AllowTcpForwarding no
```

### Références

- Documentation officielle OpenSSH : https://www.openssh.com/manual.html
- Manuel sshd_config : `man sshd_config`
- Documentation Fail2ban : https://www.fail2ban.org/wiki/index.php/MANUAL_0_8
- Guide de durcissement SSH (ANSSI) : https://www.ssi.gouv.fr
