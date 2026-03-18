# Règles techniques des casques `[v3.0]`

## RM-038h — Liste blanche des modèles VR par company

Chaque company peut restreindre les modèles de casques VR accessibles via `CompanyDeviceStandaloneType`.  
Un binaire de type `Standalone` n'est visible depuis un casque que si son `TargetDevice` est dans la liste blanche de la company.

```
Source : inversive.data/Inversive.Data/DatabaseContext.cs:319
         inversive.data/Inversive.Data/Helper/ExperienceHelper.cs
```

---

## RM-038i — Token de rafraîchissement scopé par casque

Chaque casque autonome (`DeviceStandalone`) possède son propre `RefreshToken` d'authentification.  
Ce token peut être **révoqué individuellement**.  
Il contient une date d'expiration et une date de révocation optionnelle.

```
Source : inversive.data/Inversive.Data/Models/RefreshToken.cs:18
```

---

## RM-038j — Télémétrie casque collectée en temps réel

La plateforme collecte et stocke les données de télémétrie suivantes pour chaque casque autonome :

| Donnée | Détail |
|--------|--------|
| **Batterie casque** | En % |
| **Batterie contrôleur gauche** | En % |
| **Batterie contrôleur droit** | En % |
| **Stockage primaire** | Utilisé / Maximum (Go) |
| **Stockage secondaire** | Utilisé / Maximum (Go) |
| **Température** | En °C |
| **Version firmware** | — |
| **Version Android** | — |
| **SSID Wi-Fi** | — |
| **Adresse IP** | — |
| **Adresse MAC** | — |
| **Force du signal Wi-Fi** | — |
| **LastSeen** | Horodatage de la dernière connexion |

```
Source : inversive.data/Inversive.Data/Models/DeviceStandalone.cs:26
```
