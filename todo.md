# ansible-webcluster-automation
Ce challenge a pour objectif de te faire manipuler :

 - Ansible (playbooks, inventaires, rôles)
 - Configuration de serveurs web
 - Gestion de la haute disponibilité


## Getting started
Voici les ressources qui vont te permettre de démarrer :

 - Documentation Ansible
 - Documentation du module Ansible nginx
 - Documentation HAProxy

Pour réaliser ce challenge, il te faut :

 - 3 machines (VMs ou LXC) sous Debian 12
 - Un accès SSH root ou sudo sur ces machines
 - Ansible installé sur ta machine de contrôle
 - Un accès Internet sur toutes les machines

Si tu n'as pas tout ce qu'il faut pour bien démarrer, n'hésite pas à te rapprocher de ton formateur.
Une fois que tu te sens d'attaque, tu peux démarrer ce challenge. Tu peux prendre connaissance de l'objectif ci-dessous.
Bonne pratique !

## Contexte
Tu es administrateur système junior dans une entreprise de services numériques (ESN). Un nouveau client, une agence web en pleine croissance, souhaite moderniser son infrastructure d'hébergement.
Actuellement, ils hébergent tous leurs sites web clients sur un unique serveur Apache, ce qui pose des problèmes de performances et de disponibilité. Ils souhaitent migrer vers une architecture plus robuste.
Ton responsable te confie la mission de créer une configuration Ansible réutilisable pour déployer un cluster de serveurs web avec répartition de charge.

## Objectifs
L'architecture cible doit comprendre :

 - 2 serveurs web nginx identiques
 - 1 serveur HAProxy pour la répartition de charge
 - Une page web de test simple pour valider le fonctionnement

Le client souhaite pouvoir réutiliser ta configuration pour déployer rapidement cette architecture dans différents environnements (développement, recette, production).
Il attend donc de toi, sur un dépôt gitlab que tu lui remettras à la fin de ta mission :


 - Une documentation en anglais au format markdown expliquant :
     - L'architecture mise en place
     - Les prérequis nécessaires
     - La procédure d'utilisation des playbooks
     - Les choix techniques effectués



**Un inventaire Ansible d'exemple**
 - Des playbooks Ansible pour :

     - Installer et configurer nginx sur les serveurs web
     - Installer et configurer HAProxy sur le load balancer
     - Déployer une page web de test

 - Des rôles Ansible bien structurés et réutilisables

 - Des tests basiques pour valider le bon fonctionnement


## Structure attendue
```
ansible-webcluster/
├── README.md
├── inventory/
│   └── hosts.ini
├── group_vars/
│   ├── all.yml
│   ├── webservers.yml
│   └── loadbalancer.yml
├── roles/
│   ├── nginx/
│   └── haproxy/
└── playbooks/
    ├── site.yml
    ├── webservers.yml
    └── loadbalancer.yml
```
## Critères d'évaluation  
Ta solution sera évaluée sur :

1. Fonctionnalité :

La configuration fonctionne en suivant la documentation
Le cluster web est opérationnel
La répartition de charge est effective


2. Qualité du code :

Les rôles sont bien structurés
Le code est commenté et lisible
Les bonnes pratiques Ansible sont respectées


3. Documentation :

Claire et complète
En anglais
Contient des exemples


4. Maintenabilité :

Configuration réutilisable
Variables bien organisées
Structure de projet claire




## Bonus (optionnels)
Si tu as terminé les objectifs principaux, tu peux améliorer ta solution avec :

 - Un certificat SSL auto-signé
 - Une surveillance basique avec nginx status
 - Des tests automatisés avec Ansible Molecule
 - Une gestion des sauvegardes de configuration


## Conseils

Commence par faire un plan de ton architecture
Teste d'abord manuellement les configurations
Découpe ton travail en petites étapes
Utilise git dès le début du projet
Teste régulièrement tes playbooks
Documente au fur et à mesure