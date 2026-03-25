# DHCP (Dynamic Host Configuration Protocol) - Configuration

Le service DHCP est essentiel pour gérer automatiquement les adresses IP de ton parc informatique.

---

## 🏗️ Concepts clés du DHCP

### Pourquoi automatiser ?
Imaginons un réseau avec 200 ordinateurs. Si tu dois aller sur chaque PC pour taper l'IP, le masque, la passerelle et le DNS, cela prendrait des jours et tu ferais des erreurs. Le DHCP fait tout cela en **moins d'une seconde** dès que le PC s'allume.

### L'intégration avec l'AD et le DNS
Notre DHCP est "AD-Integrated". Cela signifie qu'il travaille main dans la main avec le DNS pour enregistrer automatiquement le nom des ordinateurs qui reçoivent une IP.

---

## 1. Installation du rôle
L'installation prépare les fichiers du service sur le serveur.
![Installation DHCP](../../images/dhcp-installed-serveur01.png)

## 2. Configuration de l'Étendue (Scope)
L'étendue `tplocal` définit les règles de distribution.
![Configuration Scope DHCP](../../images/dhcp-config-server01.png)

---

## 💻 Détail des commandes (Pourquoi ces paramètres ?)

| Commande | Explication détaillée |
|:--- |:--- |
| `Add-WindowsFeature DHCP -IncludeManagementTools` | Installe le service + la console graphique. Sans `-IncludeManagementTools`, tu ne pourrais pas voir le DHCP dans tes outils d'administration. |
| `Add-DhcpServerInDC` | **Sécurité critique** : Cette commande "autorise" le serveur dans l'Active Directory. Si un serveur DHCP n'est pas autorisé, Windows le bloque pour éviter qu'un pirate ne distribue de mauvaises IP. |
| `Add-DhcpServerv4Scope` | Crée le réservoir d'IP. On commence à `.50` pour laisser les premières IP (1 à 49) disponibles pour d'autres serveurs ou imprimantes (IP statiques). |
| `Set-DhcpServerv4OptionValue` | Configure les "cadeaux" bonus envoyés avec l'IP : <br>• **Option 3** : La gateway (le routeur).<br>• **Option 6** : Les serveurs DNS (Indispensable pour l'AD).<br>• **Option 15** : Le nom du domaine (`tp.local`). |

---
