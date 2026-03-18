# Structure organisationnelle

## Hiérarchie à 3 niveaux

```
Entreprise (Company)
└── Organisation (DeviceGroupSet)  ← optionnelle, activée par module
    └── Établissement (DeviceGroup)
```

Tous les utilisateurs (hors Super Admin) sont rattachés à une **Entreprise**.  
Un **Établissement** peut exister sans être rattaché à une Organisation.

---

## RM-012 — Hiérarchie à 3 niveaux

Tous les utilisateurs (hors Super Admin) sont rattachés à une Entreprise.  
Une Entreprise peut contenir des Organisations, qui regroupent des Établissements.

---

## RM-013 — L'établissement est l'unité de base

L'établissement regroupe :

- Un ensemble d'**utilisateurs**
- Les **expériences disponibles**
- Les **appareils** (casques, bornes)

---

## RM-014 — Nomenclature personnalisable

Les termes suivants peuvent être renommés par le client pour correspondre à son vocabulaire métier :

| Terme standard | Exemple client |
|----------------|----------------|
| Établissement | Site, CHU, CHR |
| Organisation | Spécialité médicale |
| Catégorie | — |
| Ressources pédagogiques | — |

La modification s'applique **partout dans l'interface**.

---

## RM-014b — Rattachement d'un établissement modifiable à tout moment

L'affiliation d'un établissement à une organisation peut être modifiée à tout moment.  
Un établissement peut aussi exister sans organisation (« **Autre structure** »).

---

## RM-014c — Organisation optionnelle, activée par module `[v3.0]`

La hiérarchie Organisation (`DeviceGroupSet`) est une **fonctionnalité optionnelle**.  
Elle est activée si la table `Modules` contient une entrée `Name = "DeviceGroupSet"` pour la company concernée.  
Sans ce module, les établissements sont gérés **à plat** sous la company.

```
Source : inversive.data/Inversive.Data/Models/Company.cs:92
```

---

## RM-014d — Uptale conditionné par IsUptaleEnabled `[v3.0]`

Le type d'expérience **Uptale** n'est disponible pour un utilisateur que si son `DeviceGroupSet` ou son `DeviceGroup` a `IsUptaleEnabled = true`.

```
Source : inversive.bll/Inversive.BLL/GlobalRules/UserDataHelper.cs:192
```

---

## RM-014e — Types d'expériences filtrés par modules activés `[v3.0]`

Les types d'expériences accessibles sont filtrés selon les **modules contractuels** activés pour la company.  
Chaque module correspond au nom d'un `ExperienceType`.  
Les **Admins** ont accès à tous les types sans restriction.

```
Source : inversive.bll/Inversive.BLL/GlobalRules/UserDataHelper.cs:186
```

---

## Archivage

### RM-015 — Perte de données lors de l'archivage d'un établissement

!!! danger "Action irréversible"
    L'archivage d'un établissement entraîne la perte de **toutes les informations** qui lui sont liées.  
    Cette action est irréversible sans désarchivage explicite depuis la section Archives.

---

### RM-015b — Section Archives

La section Archives centralise tous les éléments archivés :

- Organisations, Établissements, Autres structures
- Expériences, Utilisateurs
- Bornes, Casques autonomes, Assets

Un élément désarchivé redevient **immédiatement disponible et valide**.
