# Lobby personnalisé `[v3.0]`

Le `CustomLobby` est l'**environnement 3D affiché sur un casque** lorsqu'aucune expérience n'est en cours.  
Il peut être personnalisé à différents niveaux de la hiérarchie.

---

## RM-065 — CustomLobby assignable à 4 niveaux

Un lobby personnalisé peut être assigné à 4 niveaux. **Le niveau le plus spécifique prend la priorité.**

| Priorité | Niveau | Entité |
|----------|--------|--------|
| 4 (plus spécifique) | Casque individuel | `DeviceStandalone` |
| 3 | Établissement | `DeviceGroup` |
| 2 | Organisation | `DeviceGroupSet` |
| 1 (plus général) | Company | `Company` |

```
Source : inversive.data/Inversive.Data/DatabaseContext.cs:212
```

---

## RM-066 — CustomLobby peut contenir des BundleAssets et des ThirdPartyApps

Un lobby personnalisé peut intégrer :

- **BundleAssets** — assets de décor, scènes 3D
- **ThirdPartyApps** — applications tierces intégrées

Ces contenus enrichissent l'environnement du lobby.
