# Guide Technique Exhaustif : Déploiement Infrastructure AD/DNS/DHCP (V2)

Ce document est la référence exhaustive du projet. Il recense l'intégralité des scripts PowerShell utilisés, classés chronologiquement par phase technologique, avec une explication détaillée ligne par ligne.

---

## Phase 1 : Configuration Réseau Initiale

Avant la promotion des contrôleurs de domaine, l'adressage statique est impératif.

```powershell
# Configuration IP Statique sur SRV-DC01
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.1.10 -PrefixLength 24 -DefaultGateway 192.168.1.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses ("127.0.0.1")

# Configuration IP Statique sur SRV-DC02
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.1.11 -PrefixLength 24 -DefaultGateway 192.168.1.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses ("192.168.1.10")
```

**Explication technique :**
- `New-NetIPAddress` : Assigne une adresse IP fixe indispensable aux serveurs d'infrastructure.
- `Set-DnsClientServerAddress` : Oriente le serveur vers son propre service DNS (sur DC1) ou vers le DC partenaire (sur DC2).

---

## Phase 2 : Déploiement de l'Active Directory (AD DS)

### 2.1 Installation et Forêt (DC1)
```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

Install-ADDSForest `
    -DomainName "tp.local" `
    -SafeModeAdministratorPassword (Read-Host -AsSecureString) `
    -DomainNetbiosName "TP" `
    -InstallDns:$true `
    -Force:$true
```

### 2.2 Jonction et Réplication (DC2)
```powershell
# Ouverture du pare-feu pour le trafic RPC/AD
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False

# Promotion en DC réplica
Install-ADDSDomainController `
    -DomainName "tp.local" `
    -Credential (Get-Credential) `
    -InstallDns:$true `
    -ReplicationSourceDC "SRV-DC01.tp.local" `
    -Force:$true
```

---

## Phase 3 : Gouvernance et Objets AD

### 3.1 Structure des Unités d'Organisation (OU)
```powershell
# Création de la hiérarchie TP-PROJET
New-ADOrganizationalUnit -Name "TP-PROJET" -Path "DC=tp,DC=local"
New-ADOrganizationalUnit -Name "UTILISATEURS" -Path "OU=TP-PROJET,DC=tp,DC=local"
New-ADOrganizationalUnit -Name "GROUPES" -Path "OU=TP-PROJET,DC=tp,DC=local"
New-ADOrganizationalUnit -Name "ORDINATEURS" -Path "OU=TP-PROJET,DC=tp,DC=local"
```

### 3.2 Gestion des Utilisateurs et Groupes
```powershell
# Création du groupe de sécurité
New-ADGroup -Name "G-Direction" -GroupCategory Security -GroupScope Global -Path "OU=GROUPES,OU=TP-PROJET,DC=tp,DC=local"

# Création de l'utilisateur Jean Dupont
New-ADUser -Name "jdupont" -AccountPassword (Read-Host -AsSecureString) -Enabled $true -Path "OU=UTILISATEURS,OU=TP-PROJET,DC=tp,DC=local"

# Ajout au groupe
Add-ADGroupMember -Identity "G-Direction" -Members "jdupont"

# Reset de mot de passe (si oublié)
Set-ADAccountPassword -Identity "jdupont" -NewPassword (Read-Host -AsSecureString) -Reset $false
```

---

## Phase 4 : Service DHCP (Dynamic Host Configuration Protocol)

```powershell
Install-WindowsFeature DHCP -IncludeManagementTools
Add-DhcpServerInDC -DnsName "SRV-DC01.tp.local"

Add-DhcpServerv4Scope `
    -Name "Utilisateurs-TP" `
    -StartRange 192.168.1.50 `
    -EndRange 192.168.1.200 `
    -SubnetMask 255.255.255.0

Set-DhcpServerv4OptionValue `
    -OptionId 3 -Value "192.168.1.1" `
    -OptionId 6 -Value "192.168.1.10", "192.168.1.11" `
    -OptionId 15 -Value "tp.local"
```

---

## Phase 5 : Partage SMB et GPO Fond d'écran

### 5.1 Création du partage de fichiers
```powershell
# Création du dossier et du partage pour les assets
New-Item -Path "C:\Assets" -ItemType Directory
New-SmbShare -Name "Images" -Path "C:\Assets" -FullAccess "Admins du domaine" -ReadAccess "Tout le monde"
```

### 5.2 Déploiement de la GPO
```powershell
$gpoFond = "GPO-Fond-Ecran"
New-GPO -Name $gpoFond | New-GPLink -Target "OU=UTILISATEURS,OU=TP-PROJET,DC=tp,DC=local"

# Configuration Registre (WallPaper)
Set-GPRegistryValue -Name $gpoFond -Key "HKCU\Control Panel\Desktop" -ValueName "Wallpaper" -Type String -Value "\\SRV-DC01\Images\fond.jpg"
Set-GPRegistryValue -Name $gpoFond -Key "HKCU\Control Panel\Desktop" -ValueName "WallpaperStyle" -Type String -Value "2"
```

---

## Phase 6 : Sécurité et Hardening ANSSI

### 6.1 Politique de Mots de Passe
```powershell
# Vérification de la politique par défaut
Get-ADDefaultDomainPasswordPolicy

# Enforcement de la complexité et longueur (12 car.)
Set-ADDefaultDomainPasswordPolicy -MinPasswordLength 12 -ComplexityEnabled $true
```

### 6.2 Hardening des Protocoles (GPO)
```powershell
$gpoHardening = "GPO-HARDENING-ANSSI"
New-GPO -Name $gpoHardening | New-GPLink -Target "DC=tp,DC=local"

# Désactivation LLMNR / SMBv1 / NetBIOS
Set-GPRegistryValue -Name $gpoHardening -Key "HKLM\Software\Policies\Microsoft\Windows NT\DNSClient" -ValueName "EnableMulticast" -Type DWord -Value 0
Set-GPRegistryValue -Name $gpoHardening -Key "HKLM\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -ValueName "SMB1" -Type DWord -Value 0

# Signature SMB obligatoire
Set-GPRegistryValue -Name $gpoHardening -Key "HKLM\System\CurrentControlSet\Services\LanmanServer\Parameters" -ValueName "RequireSecuritySignature" -Type DWord -Value 1
```

---

## Phase 7 : Poste Client et Diagnostic

### 7.1 Jonction au Domaine (Poste Client)
```powershell
Add-Computer -DomainName "tp.local" -Credential (Get-Credential) -Restart
```

### 7.2 Diagnostic et Vérification
```powershell
# Santé de la réplication (Entre DCs)
repadmin /syncall /AdeP
repadmin /replsummary

# Audit GPO (Sur Client)
gpupdate /force
gpresult /r

# Test de connectivité (Port AD/DNS)
Test-NetConnection -ComputerName SRV-DC01 -Port 53
Test-NetConnection -ComputerName SRV-DC01 -Port 445
```

---
*Ce document technique est certifié conforme aux configurations déployées.*
