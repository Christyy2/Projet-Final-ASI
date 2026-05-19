# Projet-Final-ASI
**Université :** Institut Universitaire Des Sciences (IUS)  
**Faculté :** Faculté des Sciences et des Technologies (FST)  
**Préparé par :**  
- Donsam Jean Gabard NOEL  
- Christy Gérys LAMBERT  
- Lens Sandro PETIOTE  

**Sous la Direction de :** M.Ismaël SAINT-AMOUR 

**Date :** 18 mai 2026  

---

## Table des matières

1. [Introduction](#1-introduction)
2. [Objectif du projet](#2-objectif-du-projet)
3. [Problématique](#3-problématique)
4. [Méthodologie](#4-méthodologie)
5. [Description du projet](#5-description-du-projet)
6. [Conclusion](#6-conclusion)

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

##### 1.1 Génération d’une paire de clé (publique et privé)
##### 1.2 Copie de la clé publique sur le serveur
##### 1.3 Test de la connexion par clé avant de continuer 

---

### Étape 2 — Désactivation du login root

#### Objectif

Le compte `root` est la cible principale des attaques automatisées. Le désactiver force les attaquants à deviner en plus le nom d'utilisateur, réduisant ainsi considérablement la surface d'attaque.

#### Étapes réalisées

##### 2.1 Modification du fichier sshd_config
- PermitRootlogin no
- PasswordAuthentication no
- PubkeyAuthentication yes

---

### Étape 3 — Changement du port SSH

#### Objectif

Le port 22 est scanné en permanence par des robots automatisés. Changer le port d'écoute réduit considérablement le nombre de tentatives d'intrusion visibles dans les journaux système.

#### Étapes réalisées

##### 3.1 Modification du fichier sshd_config
##### 3.2 Ouverture du nouveau port dans le pare-feu AVANT de redémarrer SSH
##### 3.3 Redémarrage du service SSH
> **Note importante :** Sur les versions récentes d'Ubuntu, SSH est géré par un **socket systemd** qui contrôle le port, et non uniquement par `sshd_config`. Si le changement de port n'a pas d'effet, il faut modifier le socket SSH :
##### 3.4 Test depuis un autre Terminal (Mon ordinateur personnel)

---

### Étape 4 — Mise en place de Fail2ban

#### Objectif

Fail2ban surveille les journaux SSH et bloque automatiquement les adresses IP qui échouent trop souvent à s'authentifier, protégeant ainsi le serveur contre les attaques par force brute.

#### Étapes réalisées

##### 4.1 Update du Serveur et Installation de Fail2ban
##### 4.2 Création du fichier de configuration local
##### 4.3 Modification de la section [sshd]
##### 4.4 Activation et démarrage de Fail2ban
##### 4.5 Vérification du statut de Fail2ban

---

### Étape 5 — Limitation des utilisateurs autorisés

#### Objectif

Restreindre les connexions SSH uniquement aux utilisateurs explicitement listés, même si d'autres comptes existent sur le serveur.

#### Étapes réalisées

##### 5.1 Modification du fichier sshd_config
##### 5.2 Tests de vérification

---

## 9. Conclusion

---


