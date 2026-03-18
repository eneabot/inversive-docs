# Règles techniques — Utilisateurs `[v3.0]`

## RM-011f — Bloqué si DeviceGroup archivé

Un utilisateur **ne peut pas se connecter** si son `DeviceGroup` associé est archivé (`IsArchived=true`), même si son propre compte est actif.

```
Source : inversive.bll/Inversive.BLL/GlobalRules/UserDataHelper.cs:150
```

---

## RM-011g — Bloqué si DeviceGroupSet archivé

Un utilisateur ne peut pas se connecter si son `DeviceGroupSet` (ou celui de son `DeviceGroup`) est archivé.

```
Source : inversive.bll/Inversive.BLL/GlobalRules/UserDataHelper.cs:152
```

---

## RM-011h — Suppression d'un DeviceGroupSet archive tous ses utilisateurs

Lorsqu'un `DeviceGroupSet` est supprimé, tous ses utilisateurs rattachés sont automatiquement :

- `IsArchived = true`
- `ArchiveDate = DateTime.UtcNow`
- Accès Keycloak verrouillé pour **100 ans**
- Cache Redis invalidé

```
Source : inversive.bll/Inversive.BLL/DataAccessor/UserDataAccessor.cs:405
```

---

## RM-011i — Désarchivage : réinitialisation de la date d'activation

Lors du désarchivage d'un utilisateur, les champs sont réinitialisés :

| Champ | Nouvelle valeur |
|-------|-----------------|
| `IsArchived` | `false` |
| `ArchiveDate` | `null` |
| `RequestedArchiveDate` | `null` |
| `ActivationDate` | `DateTime.UtcNow` |

Le verrouillage Keycloak est supprimé.

```
Source : inversive.bll/Inversive.BLL/DataAccessor/UserDataAccessor.cs:381
```

---

## RM-011j — Contenu de l'anonymisation

L'action « Anonymiser » remplace les données personnelles par des GUIDs aléatoires :

| Champ | Avant | Après |
|-------|-------|-------|
| `FirstName` | Prénom | GUID aléatoire |
| `LastName` | Nom | GUID aléatoire |
| `Email` | Email | GUID aléatoire |
| `ProducerCompany` | Valeur | `null` |
| `ProfilePicture` | Valeur | `null` |

La modification s'applique simultanément dans `AuthenticationContext` et `DatabaseContext`.

```
Source : inversive.bll/Inversive.BLL/DataAccessor/UserDataAccessor.cs:444
```

---

## RM-011k — Limite maximale des exports de données

Les exports (utilisateurs, expériences, statistiques) sont limités à **1 000 000 enregistrements** par requête.

```
Source : inversive.bll/Inversive.BLL/Helpers/SearchHelper.cs:8
```

---

## RM-011l — Email : plusieurs destinataires possibles

Le champ email peut contenir **plusieurs adresses séparées par `;`**.  
Dans ce cas, le mail est envoyé à toutes les adresses listées.

```
Source : inversive.bll/Inversive.BLL/GlobalRules/EmailHelper.cs:74
```

---

## RM-011m — Échec d'envoi email : silencieux

!!! info "Comportement"
    Un échec d'envoi d'email **n'interrompt pas** l'opération métier en cours.  
    L'erreur est capturée silencieusement (try-catch).  
    L'utilisateur est créé/modifié même si l'email n'a pas pu être envoyé.

```
Source : inversive.bll/Inversive.BLL/GlobalRules/EmailHelper.cs:79
```

---

## RM-011n — Impersonation : substitution de contexte actif

L'impersonation (`UserImpersonation`) remplace le **contexte actif** (rôle, company, DeviceGroupSet, DeviceGroup) sans modifier le profil de base.

- L'identité Keycloak de l'utilisateur réel est conservée pour l'**audit**
- Toutes les actions en mode impersonation sont **réelles et persistées**

```
Source : inversive.data/Inversive.Data/Models/UserImpersonation.cs
         inversive.bll/Inversive.BLL/GlobalRules/UserDataHelper.cs
```
