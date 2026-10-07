# Contraintes du bac à sable Cloud — P014

## 1. Environnement Cloud

Le projet P014 sera réalisé dans un environnement Cloud en utilisant les ressources disponibles dans le bac à sable AWS/Azure.

L'objectif est de concevoir une architecture réseau sécurisée avec un point d'entrée unique protégé par un Bastion Host.

## 2. Contraintes du bac à sable

Les principales contraintes identifiées sont :

* les sessions Cloud sont limitées dans le temps ;
* les ressources disponibles sont limitées ;
* les machines virtuelles doivent utiliser des tailles adaptées au projet ;
* le nombre de ressources déployées doit être maîtrisé ;
* une région Cloud doit être choisie pour le déploiement ;
* les ressources doivent être supprimées à la fin des tests afin d'éviter une consommation inutile des ressources disponibles.

## 3. Ressources prévues

L'architecture nécessite principalement :

* un réseau virtuel ;
* un sous-réseau public ;
* un sous-réseau privé ;
* un Bastion Host ;
* un ou plusieurs serveurs privés ;
* des règles de sécurité réseau ;
* une machine de test Kali Linux si elle est disponible dans le bac à sable.

## 4. Dimensionnement

Afin de respecter les limites du bac à sable, les machines virtuelles seront configurées avec des ressources minimales adaptées aux besoins du projet.

Le nombre de machines virtuelles sera limité au strict nécessaire.

Le Bastion Host et les serveurs privés utiliseront des petites instances adaptées aux besoins de configuration, de connexion SSH et de tests.

## 5. Impact sur l'architecture

Les contraintes du bac à sable imposent une architecture simple et optimisée.

Le réseau sera segmenté en plusieurs zones afin de respecter le principe de sécurité du projet :

* zone publique ;
* DMZ contenant le Bastion Host ;
* réseau privé contenant les ressources protégées.

Le Bastion constituera le point d'entrée contrôlé vers le réseau privé.

## 6. Gestion des sessions

Les ressources Cloud seront utilisées uniquement pendant les phases nécessaires :

1. création de l'infrastructure ;
2. configuration du réseau ;
3. configuration du Bastion ;
4. configuration des serveurs privés ;
5. réalisation des tests ;
6. collecte des preuves ;
7. suppression ou arrêt des ressources après les tests.

Cette organisation permet de limiter l'utilisation des ressources du bac à sable.
