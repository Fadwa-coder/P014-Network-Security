# P014 : Contraintes du sandbox et dimensionnement des ressources


---

## 1. Introduction

Dans le cadre du projet **P014**, l'objectif est de concevoir un réseau segmenté avec un **point d'entrée unique et sécurisé (Bastion Host)**.

L'environnement de déploiement est un sandbox cloud aux ressources limitées. Ce document identifie les contraintes de cet environnement, dimensionne les ressources en conséquence et consigne les implications du caractère éphémère sur l'architecture.

---

## 2. Contraintes du sandbox

| Contrainte | Valeur | Conséquence principale |
|---|---|---|
| Durée d'une session | 1 h, éphémère | Déploiement et tests rapides |
| Budget d'heures mensuel | [À COMPLÉTER] h / mois | Nombre de sessions limité |
| Types de VM | t3.micro, t3.small (AWS) | Machines de petite taille uniquement |
| Région | [À COMPLÉTER] (région unique) | Toute l'architecture dans une seule région |
| Fin de session | Suppression de toutes les ressources | Environnement à reconstruire à chaque session |

### 2.1 Sessions éphémères

Les sessions sont limitées à environ **1 heure**. Les ressources doivent donc être déployées, configurées et testées rapidement.

### 2.2 Ressources limitées

Le sandbox fournit des instances de petite taille : `t3.micro` et `t3.small`. L'objectif est d'éviter de déployer des machines inutilement puissantes.

### 2.3 Région cloud

Les ressources sont déployées dans la région imposée par le sandbox : **[À COMPLÉTER]**. L'utilisation d'une seule région simplifie la communication entre les ressources et évite une architecture inutilement complexe.

### 2.4 Suppression des ressources

Les ressources sont supprimées à la fin de la session. L'architecture doit donc être **reproductible**, et les configurations importantes doivent être documentées et scriptées.

### 2.5 Budget d'heures mensuel

- Budget mensuel : **[À COMPLÉTER] heures**, soit environ [À COMPLÉTER] sessions de 1 h.
- Répartition prévue :
  - [À COMPLÉTER] session(s) de déploiement et de mise au point
  - [À COMPLÉTER] session(s) de tests de sécurité
  - [À COMPLÉTER] session(s) de marge pour les imprévus
- Chaque heure consommée est décomptée du budget : aucune machine ne doit rester allumée inutilement.

---

## 3. Dimensionnement des ressources

Pour démontrer le principe de sécurité du projet, une architecture minimale est suffisante.

### 3.1 Ressources principales

| Ressource | Rôle | Dimensionnement |
|---|---|---|
| Bastion Host | Point d'entrée SSH unique | 1 × t3.micro |
| Serveur privé | Serveur protégé | 1 × t3.micro |
| Client / Kali Linux | Tests de sécurité | 1 × t3.small, allumée uniquement pendant les tests |
| VPC | Réseau isolé | 1 VPC |
| Sous-réseau public | Zone du Bastion (rôle de DMZ) | 1 sous-réseau |
| Sous-réseau privé | Zone du serveur privé | 1 sous-réseau |
| Security Groups | Contrôle du trafic (rôle de pare-feu) | Règles restrictives, refus par défaut |

### 3.2 Dimensionnement détaillé et justification

| Machine | Type | vCPU / RAM | Justification |
|---|---|---|---|
| Bastion Host | t3.micro | 2 / 1 Go | Service SSH uniquement, charge très faible |
| Serveur privé | t3.micro | 2 / 1 Go | Ressource à protéger, aucun service lourd |
| Kali Linux | t3.small | 2 / 2 Go | Les outils de scan (`nmap`, etc.) demandent plus de mémoire |

- **Nombre d'instances :** 3 au maximum (2 en permanence, Kali pendant les tests).
- **Pare-feu :** assuré par les Security Groups AWS, sans VM dédiée, ce qui économise une instance et du budget.
- **Région :** [À COMPLÉTER], imposée par le sandbox.
- **Correspondance avec la note de cadrage :** le sous-réseau public contenant le Bastion joue le rôle de la DMZ, et les Security Groups remplacent la VM pare-feu prévue en v1.0.

### 3.3 Pourquoi ce dimensionnement

Deux petites instances permanentes, plus Kali ponctuelle, suffisent pour démontrer le Bastion et l'isolation du serveur privé. Ce choix permet de :

- réduire la consommation du budget d'heures ;
- respecter les limites du sandbox ;
- démarrer rapidement les machines ;
- faciliter les tests ;
- reconstruire facilement l'environnement.

Il n'est pas nécessaire d'utiliser davantage de serveurs pour démontrer le principe du Bastion.

---

## 4. Architecture proposée

```text
                    INTERNET
                       |
                       |
              [ Sous-réseau public ]
                       |
                  [ BASTION ]
                  SSH - Port 22
                       |
                       |
              [ Sous-réseau privé ]
                       |
                [ SERVEUR PRIVÉ ]
```

Le Bastion est le **seul point d'entrée** vers le réseau privé. Le serveur privé n'est pas directement accessible depuis Internet.

---

## 5. Principe de sécurité

Le réseau suit le principe du **moindre privilège** (*Least Privilege*).

| Flux | Règle |
|---|---|
| Internet → Bastion | SSH (22/TCP) uniquement, limité à l'IP source autorisée (pas `0.0.0.0/0`) |
| Bastion → Serveur privé | SSH (22/TCP) uniquement |
| Internet → Serveur privé | Refusé |
| Tout autre flux | Refusé (règle par défaut) |

```text
Internet
   |
   | SSH : 22 (IP autorisée)
   v
Bastion
   |
   | SSH : 22 (depuis le Security Group du Bastion)
   v
Serveur privé
```

Sur le serveur privé, la règle d'entrée SSH référence le **Security Group du Bastion** plutôt qu'une plage d'adresses : seul le Bastion peut s'y connecter.

### 5.1 Durcissement minimal du Bastion

- Authentification SSH par clé uniquement, mots de passe désactivés.
- Connexion directe en `root` désactivée.
- Comptes nominatifs avec droits minimaux.
- Journalisation des connexions.

---

## 6. Implications du caractère éphémère

Le caractère temporaire du sandbox influence directement l'architecture.

| Contrainte | Conséquence | Décision d'architecture |
|---|---|---|
| Ressources supprimées en fin de session | Rien ne persiste | Déploiement scripté (AWS CLI, Terraform ou CloudFormation) et versionné dans GitHub |
| Session de 1 h | Temps limité | Déploiement visé en moins de [À COMPLÉTER] min, le reste du temps consacré aux tests |
| Adresses IP publiques changeantes | Plan d'adressage non stable | Plan d'adressage privé fixé dans les scripts, IP publique du Bastion relevée à chaque session |
| Clés SSH perdues | Pas de réutilisation | Nouvelle paire de clés générée à chaque déploiement |
| Logs et preuves effacés | Résultats perdus | Export des captures, sorties `nmap` et logs vers GitHub avant la fin de chaque session |
| Pas d'accès Internet depuis le sous-réseau privé | Installation de paquets impossible | Installer les paquets nécessaires dans le script de démarrage ou prévoir un NAT si disponible |
| Budget d'heures limité | Peu d'essais possibles | Scripts préparés et relus hors session |

### 6.1 Déroulé type d'une session de 1 h

| Phase | Durée indicative |
|---|---|
| Déploiement automatisé | 10 min |
| Vérification de l'infrastructure | 5 min |
| Tests de sécurité (Kali) | 30 min |
| Export des preuves et logs | 10 min |
| Marge | 5 min |

---

## 7. Tests prévus

Une fois l'architecture déployée, les tests suivants sont réalisés.

### Test 1 : Accès au Bastion

Vérifier qu'un client autorisé peut se connecter au Bastion en SSH.

```bash
ssh -i cle.pem utilisateur@IP_BASTION
```

**Résultat attendu :** connexion réussie.

### Test 2 : Accès au serveur privé depuis le Bastion

```bash
ssh utilisateur@IP_SERVEUR_PRIVE
```

**Résultat attendu :** connexion réussie.

### Test 3 : Accès direct au serveur privé

Depuis Internet ou un client externe (Kali), vérifier que le serveur privé n'est **pas joignable directement**.

**Résultat attendu :** connexion refusée ou expirée.

### Test 4 : Scan de ports

Depuis Kali, scanner le Bastion et le réseau privé.

```bash
nmap -Pn IP_BASTION
nmap -Pn IP_SERVEUR_PRIVE
```

**Résultat attendu :** seul le port 22 du Bastion est visible, aucun port ouvert sur le serveur privé.

### Test 5 : Authentification non autorisée

Tenter une connexion par mot de passe et en `root` sur le Bastion.

**Résultat attendu :** connexion refusée.

Chaque test est documenté avec : objectif, commande, résultat attendu, résultat obtenu, preuve (capture ou log).

---

## 8. Résultat attendu

```text
             INTERNET
                 |
                 v
        +----------------+
        |    BASTION     |
        |   SSH : 22     |
        +----------------+
                 |
                 | accès contrôlé
                 v
        +----------------+
        | RÉSEAU PRIVÉ   |
        |                |
        | SERVEUR PRIVÉ  |
        +----------------+
```

Le Bastion constitue le **point d'entrée unique** vers le réseau privé. Le serveur privé est isolé d'Internet et accessible uniquement selon les règles de sécurité définies.

---

## 9. Points à valider avec l'encadrant

- [ ] Budget d'heures mensuel et région confirmés
- [ ] Pare-feu réalisé avec les Security Groups (pas de VM dédiée)
- [ ] Outil de déploiement automatisé autorisé (Terraform, CloudFormation ou AWS CLI)
- [ ] Mise à jour de la note de cadrage en v1.1 (passage de VMware à AWS)

---

## 10. Conclusion

Le dimensionnement proposé privilégie une architecture **simple, légère, sécurisée et reproductible**. Il respecte les contraintes du sandbox (sessions de 1 h, budget d'heures, petites instances, région unique, suppression en fin de session) tout en mettant en œuvre les principes du projet P014 :

- Segmentation réseau
- Bastion Host
- Point d'entrée unique
- Moindre privilège
- Isolation du serveur privé
- Contrôle des accès
- Reproductibilité de l'environnement
