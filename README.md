# Infrastructure Windows Server - AD / DNS / DHCP

## Introduction
Ce projet documente la mise en place d'une infrastructure réseau sécurisée basée sur Windows Server 2022. L'objectif est de fournir un service d'annuaire (Active Directory) hautement disponible, une résolution de noms (DNS) et une attribution d'adresses dynamique (DHCP).

---

# Projet Infrastructure Windows Server - AD / DNS / DHCP

Bienvenue dans le rapport complet de déploiement de l'infrastructure réseau `tp.local`. Ce projet simule un environnement d'entreprise réel avec deux contrôleurs de domaine redondants et une gestion centralisée des postes clients.

![Aperçu Infrastructure](./images/serveur01-adforestinstalled-redemarrage.png)

## 📋 État du Projet
| Composant | Status | Description |
| :--- | :--- | :--- |
| **SRV-DC01** | ✅ Opérationnel | Maître d'opérations, DNS, DHCP |
| **SRV-DC02** | ✅ Opérationnel | Réplica, Haute Disponibilité |
| **Poste Client** | ✅ Intégré | Windows 10 joint au domaine `tp.local` |
| **Gouvernance** | ✅ Configuré | OUs, Groupes, Sécurité ANSSI (12 char.) |

---

## Navigation dans la Documentation

Pour une exploration détaillée de chaque phase, veuillez consulter les guides ci-dessous :

### 1. [Infrastructure Active Directory](./docs/active-directory/README.md)
*Promotion des contrôleurs, réplication et état de santé de l'annuaire.*

### 2. [Service DHCP & Réseau](./docs/dhcp/README.md)
*Automatisation de l'adressage IP et configuration des options de portée.*

### 3. [Intégration Poste Client](./docs/poste-client/README.md)
*Jonction au domaine et gestion des sessions utilisateurs.*

### 4. [Sécurité & Hardening](./docs/securite/README.md)
*Politiques de mots de passe (ANSSI) et gestion des permissions d'accès.*

### 5. [Validation & Troubleshooting](./docs/validation/README.md)
*Preuves de conformité et procédures de reprise après incident.*

---

## Guide Technique Rapide
Pour une consultation rapide des syntaxes PowerShell et des paramètres réseau, veuillez vous référer au **[GUIDE TECHNIQUE CENTRALISÉ](./docs/TECHNICAL_GUIDE.md)**.

---
*Projet réalisé dans le cadre du TP Réseau - Mars 2026*
