# 📘 Guide Technique : Comprendre l'Infrastructure AD / DNS / DHCP

Ce document est ton compagnon pour comprendre le "Pourquoi" derrière chaque clic et chaque commande.

---

## 1. AD DS : Le Cœur de l'Identité
L'**Active Directory Domain Services** n'est pas juste une liste d'utilisateurs. C'est une base de données hiérarchique qui gère la **confiance**.
- **Pourquoi `tp.local` ?** : Le `.local` est une convention de lab. En entreprise, on utiliserait un sous-domaine comme `ad.entreprise.com`.
- **Pourquoi 2 DCs ?** : Si `SRV-DC01` brûle, `SRV-DC02` prend le relais instantanément. C'est la **Haute Disponibilité**.

---

## 2. DNS : L'Aiguilleur du Ciel
Le DNS transforme les noms (ex: `tp.local`) en adresses IP.
- **Le secret de l'AD** : L'AD utilise des enregistrements cachés appelés **SRV**. Sans eux, un PC Windows ne sait pas à quel serveur envoyer ton mot de passe.
- **Commandes clés** : 
  - `Resolve-DnsName` : La version moderne de PowerShell pour tester le DNS.
  - `nslookup` : L'outil universel (Windows, Linux, Mac) pour diagnostiquer le DNS.

---

## 3. DHCP : Le Distributeur Automatique
Le DHCP évite les conflits d'IP (quand deux PC ont la même adresse).
- **Le "Bail" (Lease)** : Une IP n'est pas donnée pour toujours. Le client la "loue" pour 8 jours par défaut.
- **Les Options (3, 6, 15)** : Ce sont des standards mondiaux (RFC). 
  - Le chiffre `6` signifiera TOUJOURS "Serveur DNS" pour n'importe quel appareil dans le monde.

---

## 4. GPO (Group Policy Object) : La Loi du Domaine
Les GPO sont des regroupements de paramètres que tu "pousses" sur tes machines et utilisateurs.
- **Lien (Link)** : Une GPO ne fait rien tant qu'elle n'est pas "liée" à une Unité d'Organisation (OU).
- **GPUpdate** : Par défaut, les PC vérifient les nouvelles GPO toutes les 90 minutes. `gpupdate /force` permet de le faire immédiatement.

---

## 💻 Lexique des Commandes PowerShell (Deep Dive)

| Commande | Pourquoi on l'utilise ? | Que se passe-t-il si on ne la fait pas ? |
|:--- |:--- |:--- |
| `Install-ADDSForest` | Initialise tout le système. | Pas de domaine, pas d'AD, juste un serveur seul. |
| `Install-ADDSDomainController` | Relie deux serveurs entre eux. | Pas de secours en cas de panne. |
| `Add-DhcpServerInDC` | Active la sécurité DHCP. | Le serveur restera "Eteint" et refusera de donner des IP. |
| `Set-NetFirewallProfile` | Ouvre les portes du serveur. | Les autres machines seront bloquées et ne pourront pas communiquer. |
| `Disable-NetAdapterBinding` | Désactive l'IPv6. | Optionnel, mais évite que l'IPv6 ne prenne la priorité sur nos tests IPv4. |

---
