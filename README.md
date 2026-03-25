# 📂 Infrastructure Windows Server - AD / DNS / DHCP

## Elif JAFFRES

## 📖 Introduction
Ce projet documente la mise en place d'une infrastructure réseau sécurisée basée sur Windows Server 2022. L'objectif est de fournir un service d'annuaire (Active Directory) hautement disponible, une résolution de noms (DNS) et une attribution d'adresses dynamique (DHCP).

---

# Projet Infrastructure Windows Server - AD / DNS / DHCP

Le rapport complet de déploiement de l'infrastructure réseau `tp.local`. Ce projet simule un environnement d'entreprise réel avec deux contrôleurs de domaine redondants et une gestion centralisée des postes clients.

![Aperçu Infrastructure](./images/serveur01-adforestinstalled-redemarrage.png)

## États
| Composant | Status | Description |
| :--- | :--- | :--- |
| **SRV-DC01** | ✅ Opérationnel | Maître d'opérations, DNS, DHCP |
| **SRV-DC02** | ✅ Opérationnel | Réplica, Haute Disponibilité |
| **Poste Client** | ✅ Intégré | Windows 10 joint au domaine `tp.local` |
| **Gouvernance** | ✅ Configuré | OUs, Groupes, Sécurité ANSSI (12 char.) |

---

## Navigation dans la Documentation

Pour une exploration détaillée de chaque phase, utilisez les guides ci-dessous :

### 1. [Infrastructure Active Directory](./docs/active-directory/README.md)
*Promotion des contrôleurs, réplication et santé de l'annuaire.*

### 2. [Service DHCP & Réseau](./docs/dhcp/README.md)
*Automatisation de l'adressage IP et options de portée.*

### 3. [Intégration Poste Client](./docs/poste-client/README.md)
*Jonction au domaine et expérience utilisateur.*

### 4. [Sécurité & Hardening](./docs/securite/README.md)
*Politiques de mots de passe (ANSSI) et gestion des permissions ACL.*

### 5. [Validation & Troubleshooting](./docs/validation/README.md)
*Preuves de succès et résolution des incidents post-crash.*

---

## Guide Technique Rapide
Si vous avez besoin de vérifier une commande ou un paramètre réseau rapidement, consultez le **[GUIDE TECHNIQUE CENTRALISÉ](./docs/TECHNICAL_GUIDE.md)**.


