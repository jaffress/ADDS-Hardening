## 💡 Pourquoi le DNS est-il lié à l'AD ?

L'Active Directory ne "connaît" pas les adresses IP. Il utilise le DNS pour que les clients trouvent les serveurs. 
- Sans DNS, un ordinateur ne pourrait pas savoir que `SRV-DC01` est le serveur de connexion.
- **Enregistrements SRV** : Ce sont des "balises" dans le DNS qui disent "Le service d'annuaire LDAP se trouve sur telle IP".

---

## 💻 Commandes DNS utilisées

| Commande | À quoi ça sert ? |
|:--- |:--- |
| `Resolve-DnsName tp.local` | Teste si le nom du domaine est correctement transformé en adresse IP. |
| `nslookup tp.local` | Outil classique pour interroger le serveur DNS et vérifier sa réponse. |
| `Set-DnsClientServerAddress` | Configure la carte réseau pour qu'elle interroge le bon serveur DNS. |

---
