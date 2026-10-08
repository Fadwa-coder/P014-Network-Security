# Note de cadrage — Projet P014

## 1. Présentation du projet

Le projet consiste à concevoir une architecture réseau sécurisée
et segmentée afin de protéger les ressources privées contre les
accès non autorisés.

L'architecture repose sur un point d'entrée unique et sécurisé :
le Bastion Host.

## 2. Objectif général

Concevoir et mettre en place un réseau segmenté avec un point
d'entrée unique protégé afin de limiter les accès directs aux
ressources du réseau privé.

## 3. Objectifs spécifiques

- Séparer les différentes zones du réseau.
- Mettre en place une zone DMZ.
- Déployer un Bastion Host comme point d'entrée unique.
- Protéger les serveurs du réseau privé.
- Contrôler les communications avec un pare-feu.
- Appliquer le principe du moindre privilège.
- Tester la sécurité de l'architecture avec Kali Linux.
- Documenter les résultats des tests.

## 4. Architecture prévue

L'architecture comprendra :

- une zone publique ;
- un pare-feu assurant le contrôle des communications ;
- une zone DMZ ;
- un Bastion Host comme point d'entrée unique ;
- un pare-feu assurant le contrôle des communications ;
- un réseau privé isolé ;
- des serveurs privés ;
- une machine Kali Linux pour les tests de sécurité.

Le pare-feu contrôlera les communications entre les différentes
zones du réseau.

Le Bastion constituera le point d'entrée contrôlé vers les
ressources du réseau privé.

Les serveurs privés ne seront pas directement accessibles depuis
la zone publique.
## 5. Technologies utilisées

- VMware
- Ubuntu Server
- Ubuntu Desktop
- Kali Linux
- SSH
- Pare-feu
- Git
- GitHub
- PlantUML

## 6. Critères de réussite

Le projet sera considéré comme réussi si :

- le réseau public et le réseau privé sont séparés ;
- le serveur privé n'est pas directement accessible depuis
  le réseau public ;
- le Bastion constitue le point d'entrée contrôlé ;
- les accès SSH sont limités aux flux autorisés ;
- les communications non autorisées sont bloquées par le pare-feu ;
- les tests de sécurité avec Kali Linux sont réalisés ;
- les résultats des tests sont documentés ;
- la configuration du projet peut être reproduite.

## 7. Tests prévus

Les tests porteront notamment sur :

- la connectivité réseau ;
- l'accès SSH au Bastion ;
- l'accès du Bastion au serveur privé ;
- le blocage d'un accès direct au serveur privé ;
- le fonctionnement des règles du pare-feu ;
- les tentatives d'accès non autorisées.

## 8. Contraintes

Le projet sera réalisé dans un environnement de virtualisation
local afin de limiter les coûts.

Les ressources disponibles sur la machine hôte détermineront
le nombre de machines virtuelles utilisées.

## 9. Résultat attendu

À la fin du projet, une architecture réseau segmentée et sécurisée
devra être opérationnelle, testée et documentée.

Le projet devra démontrer qu'un utilisateur externe ne peut pas
accéder directement aux ressources privées et que les accès
autorisés passent par le point d'entrée sécurisé.
