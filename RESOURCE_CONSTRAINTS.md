# P014 — Contraintes du Sandbox et Dimensionnement des Ressources

## 1. Introduction

Dans le cadre du projet **P014 — Design a Network a Hacker Can't Just Walk Into**, l'objectif est de concevoir un réseau segmenté avec un **point d'entrée unique et sécurisé (Bastion Host)**.

L'environnement de déploiement utilise un sandbox cloud avec des ressources limitées. Il est donc nécessaire d'identifier les contraintes de l'environnement et de dimensionner les ressources afin de conserver une architecture simple, sécurisée et adaptée aux limites du sandbox.

---

## 2. Contraintes du Sandbox

Les principales contraintes identifiées sont les suivantes :

### 2.1 Sessions éphémères

Les sessions du sandbox sont limitées dans le temps, avec des sessions pouvant être d'environ **1 heure**.

Cette contrainte signifie que les ressources doivent être configurées et testées rapidement.

### 2.2 Ressources limitées

Le sandbox fournit des ressources de petite taille adaptées aux travaux pratiques.

Pour AWS, des instances telles que :

* `t3.micro`
* `t3.small`

peuvent être utilisées selon les ressources disponibles dans le sandbox.

L'objectif est d'éviter de déployer des machines inutilement puissantes.

### 2.3 Région cloud

Les ressources doivent être déployées dans une région cloud disponible pour le sandbox.

L'utilisation d'une même région permet de simplifier la communication entre les différentes ressources et d'éviter une architecture inutilement complexe.

### 2.4 Suppression des ressources

Les ressources du sandbox peuvent être supprimées à la fin de la session.

L'architecture doit donc être facilement **reproductible**.

Les configurations importantes doivent être documentées afin de pouvoir reconstruire l'environnement rapidement.

---

# 3. Dimensionnement des ressources

Pour démontrer le principe de sécurité du projet P014, une architecture minimale est suffisante.

## 3.1 Ressources principales

| Ressource       | Rôle                      | Dimensionnement proposé              |
| --------------- | ------------------------- | ------------------------------------ |
| Bastion Host    | Point d'entrée SSH unique | AWS t3.micro                         |
| Serveur privé   | Serveur protégé           | AWS t3.micro                         |
| VPC             | Réseau isolé              | 1 VPC                                |
| Subnet public   | Zone du Bastion           | 1 subnet public                      |
| Subnet privé    | Zone du serveur           | 1 subnet privé                       |
| Security Groups | Contrôle du trafic        | Règles restrictives                  |
| Client / Kali   | Tests de sécurité         | Utilisé uniquement pendant les tests |

---

# 4. Architecture proposée

L'architecture retenue est basée sur le principe :

```text
                    INTERNET
                       |
                       |
                 [ Subnet Public ]
                       |
                  [ BASTION ]
                  SSH - Port 22
                       |
                       |
              [ Subnet Privé ]
                       |
                [ SERVEUR PRIVÉ ]
```

Le Bastion représente le **seul point d'entrée** vers le réseau privé.

Le serveur privé n'est pas directement accessible depuis Internet.

---

# 5. Principe de sécurité

Le réseau suit le principe du **Least Privilege**.

### Accès Internet

Internet ne doit pas pouvoir accéder directement au serveur privé.

### Accès au Bastion

Le Bastion accepte uniquement les connexions nécessaires, principalement **SSH sur le port 22**.

### Accès au serveur privé

Le serveur privé accepte uniquement les connexions provenant du Bastion.

Exemple :

```text
Internet
   |
   | SSH : 22
   v
Bastion
   |
   | SSH : 22
   v
Serveur privé
```

Le serveur privé ne possède donc pas de point d'accès direct depuis Internet.

---

# 6. Justification du dimensionnement

Le choix de deux petites instances permet de respecter les contraintes du sandbox tout en démontrant les principaux concepts de cybersécurité du projet.

L'utilisation de machines de petite taille permet :

* de réduire la consommation des ressources ;
* de limiter les coûts ;
* de respecter les limites du sandbox ;
* de démarrer rapidement les machines ;
* de faciliter les tests ;
* de reconstruire facilement l'environnement.

Il n'est pas nécessaire d'utiliser plusieurs serveurs pour démontrer le principe du Bastion.

---

# 7. Implications du caractère éphémère

Le caractère temporaire du sandbox influence directement l'architecture.

Les ressources doivent être :

1. **simples à déployer ;**
2. **rapides à configurer ;**
3. **faciles à tester ;**
4. **faciles à supprimer ;**
5. **faciles à reconstruire.**

Les configurations réseau, les règles de sécurité et les paramètres SSH doivent donc être documentés.

Une architecture minimale est privilégiée afin de réduire le temps nécessaire à la reconstruction de l'environnement.

---

# 8. Tests prévus

Une fois l'architecture déployée, les tests suivants seront réalisés :

### Test 1 — Accès au Bastion

Vérifier qu'un client autorisé peut se connecter au Bastion avec SSH.

```bash
ssh utilisateur@IP_BASTION
```

### Test 2 — Accès au serveur privé

Depuis le Bastion, vérifier que le serveur privé est accessible :

```bash
ssh utilisateur@IP_SERVEUR_PRIVE
```

### Test 3 — Accès direct au serveur privé

Depuis Internet ou un client externe, vérifier que le serveur privé n'est **pas directement accessible**.

Résultat attendu :

```text
Client externe ---> Serveur privé
                  ❌ ACCÈS REFUSÉ
```

### Test 4 — Accès via Bastion

```text
Client ---> Bastion ---> Serveur privé
             ✅             ✅
```

L'accès doit être autorisé uniquement lorsque le chemin prévu par l'architecture est respecté.

---

# 9. Résultat attendu

L'architecture finale doit respecter le principe suivant :

```text
             INTERNET
                 |
                 v
        +----------------+
        |    BASTION     |
        |  SSH : 22      |
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

Le Bastion constitue ainsi le **point d'entrée unique** vers le réseau privé.

Le serveur privé est isolé d'Internet et accessible uniquement selon les règles de sécurité définies.

---

# 10. Conclusion

Le dimensionnement proposé privilégie une architecture **simple, légère, sécurisée et reproductible**.

Deux instances de petite taille sont suffisantes pour démontrer le fonctionnement du Bastion et l'isolation du serveur privé.

Cette approche permet de respecter les contraintes du sandbox tout en mettant en œuvre les principes fondamentaux du projet P014 :

* **Segmentation réseau**
* **Bastion Host**
* **Point d'entrée unique**
* **Least Privilege**
* **Isolation du serveur privé**
* **Contrôle des accès**
* **Reproductibilité de l'environnement**
