# Architecture Globale

## Vue d'ensemble réseau

### Schéma logique

```
┌─────────────────────────────────────────┐
│         Domaine: tp.local               │
│                                         │
│  ┌──────────────────────────────────┐  │
│  │ Server-01 (DC1 - Primaire)      │  │
│  │ IP: 192.168.1.10                │  │
│  │ Rôles:                          │  │
│  │ - AD DS                         │  │
│  │ - DNS (zone primaire)           │  │
│  │ - DHCP                          │  │
│  └──────────────────────────────────┘  │
│           │                             │
│           │ (Réplication AD)            │
│           │                             │
│  ┌──────────────────────────────────┐  │
│  │ Server-02 (DC2 - Réplica)       │  │
│  │ IP: 192.168.1.11                │  │
│  │ Rôles:                          │  │
│  │ - AD DS (réplica)               │  │
│  │ - DNS (zone secondaire)         │  │
│  └──────────────────────────────────┘  │
│                                         │
└─────────────────────────────────────────┘
         │
         │ (Groupe Workgroup - Client)
         │
    ┌────────────────┐
    │ Client-1       │
    │ IP: DHCP       │
    │ OS: Windows 10 │
    │ Domaine: tp    │
    └────────────────┘
```

---

## Adressage IP

| Machine | IP Statique | Rôle | Remarques |
|---------|-------------|------|-----------|
| Server-01 (DC1) | 192.168.1.10 | Contrôleur de domaine primaire | Serveur DHCP |
| Server-02 (DC2) | 192.168.1.11 | Contrôleur de domaine réplica | Serveur DHCP de secours |
| Client-1 | DHCP | Client du domaine | Reçoit 192.168.1.100+ |

### Plage DHCP

- **Étendue** : 192.168.1.50 - 192.168.1.200
- **Exclusions** : 192.168.1.1 - 192.168.1.49 (DC, passerelle, réservés)
- **Passerelle** : 192.168.1.1
- **DNS Primaire** : 192.168.1.10 (DC1)
- **DNS Secondaire** : 192.168.1.11 (DC2)

---

## Domaine Active Directory

- **Nom de domaine** : `tp.local`
- **Mode fonctionnel** : 2016 minimum (Windows Server 2022 par défaut)
- **Forêt** : Monofôret
- **Zone DNS** : Intégrée Active Directory

---

## Services configurés

### 1. Active Directory Domain Services (AD DS)
- **DC1** : Rôle de contrôleur de domaine primaire (FSMO)
- **DC2** : Réplica - tous les rôles FSMO transférables

### 2. DNS (Domain Name System)
- Zones intégrées AD : `tp.local` forward
- Zone reverse (dépend besoin)
- Résolution pour DC1 et DC2

### 3. DHCP
- Scope unique : 192.168.1.50 - 192.168.1.200
- Configuration :
  - Bail : 8 jours
  - Passerelle : 192.168.1.1
  - Serveurs DNS : DC1, DC2

---

## Redondance et HA

- **AD** : Réplication multi-maître (DC1 ↔ DC2)
- **DNS** : Zone primaire sur DC1, secondaire sur DC2
- **DHCP** : Configuration sur DC1 (DC2 de secours si besoin)
- **Failover** : En cas de perte DC1, Client utilise DC2

---

## Sécurité

- Mots de passe DSRM (Directory Services Restore Mode)
- Firewall Windows configuré
- GPO de sécurité ANSSI
- Audit des logons/modifications AD
- SMBv1 désactivé

---

## Flux de communication

```
Client-1 (DHCP) ──→ DC1:67/UDP (DHCP)
                 ←── IP + DNS (192.168.1.10, 192.168.1.11)

Client-1        ──→ DC1:53/UDP (DNS query)
                 ←── Réponse A record

Client-1        ──→ DC1:88 (Kerberos)
                 ←── Authentification + ticket

DC1 ←→ DC2 (Port 135, 139, 445, 3389) - Réplication AD
```

---

---

## Rationnel de l'architecture

### Pourquoi deux Contrôleurs de Domaine (DC1 & DC2) ?
- **Haute Disponibilité** : Si `SRV-DC01` tombe en panne, `SRV-DC02` prend le relais pour l'authentification et DNS. Pas d'arrêt de service pour les utilisateurs.
- **Équilibrage de charge** : Les requêtes DNS et d'authentification peuvent être réparties entre les deux serveurs.

### Pourquoi un domaine .local ?
- **Isolation** : Utiliser un suffixe non routable sur Internet (`.local` ou `.lan`) permet de bien séparer le réseau interne de l'Internet public, évitant des conflits de noms accidentels.

### Pourquoi DHCP sur le DC ?
- **Intégration DNS** : Le serveur DHCP Windows peut mettre à jour automatiquement les enregistrements DNS des clients lorsqu'ils reçoivent une IP, ce qui simplifie grandement la gestion des machines sur le domaine.

---

## Point clé

L'infrastructure est **monofôret, monodomain** avec **redondance par réplication AD** sur le second DC. 
Un contrôleur de domaine fait aussi rôles : DNS + DHCP.
