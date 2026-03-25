# Rapport de Vérification Post-Incident

Ce document permet de vérifier l'état des services après une coupure inattendue des VMs.

## 1. État de l'Annuaire (AD)

### Vérification de l'OU et de l'Utilisateur
```powershell
# Vérifie si l'OU TP-PROJET existe
Get-ADOrganizationalUnit -Filter "Name -eq 'TP-PROJET'"
# Vérifie si Jean Dupont est là
Get-ADUser -Identity "jdupont" -Properties MemberOf
```

### Vérification du Groupe
```powershell
Get-ADGroup -Identity "G-DIRECTION"
```

## 2. État des Politiques (GPO)

```powershell
# Liste les GPOs créées
Get-GPO -Name "GPO-Securite", "GPO-Fond-Ecran"
# Vérifie les liens sur l'OU
Get-GPInheritance -Target "OU=TP-PROJET,DC=tp,DC=local"
```

## 3. État du Client
```powershell
# Vérifie si Client-1 est toujours dans l'OU ORDINATEURS
Get-ADComputer -Filter "Name -like 'DESKTOP*'" -Properties CanonicalName | Select-Object Name, CanonicalName
```

## 4. État de la Réplication
```powershell
repadmin /replsummary
```

---

## Résultat de l'Audit :
- OUs présentes : [ ]
- Utilisateur jdupont : [ ]
- GPOs actives : [ ]
- Réplication OK : [ ]
