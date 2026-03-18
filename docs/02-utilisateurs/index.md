# Utilisateurs & Rôles

## Les 6 rôles de la plateforme

Du plus étendu au plus restreint :

| Rôle métier | Terme technique | `BaseUserType` | Description |
|---|---|---|---|
| **Super Admin** | `Admin` | `1` | Employé Inversive. Pas de company. Peut créer des clients et mettre à jour les apps. |
| **Administrateur** | `CompanyOwner` | `2` | Droits les plus étendus côté client. Accès complet à sa company. |
| **Gestionnaire d'organisation** | `DeviceGroupSetManager` | `4` | Gère un parc restreint, crée établissements/apprenants/expériences dans son org. |
| **Gestionnaire d'établissement** | `DeviceGroupManager` | `8` | Crée apprenants et expériences, uniquement dans son établissement. |
| **Apprenant** | `User` | `16` | Accède au catalogue et à ses statistiques personnelles. |
| **Producteur de contenu** | `Producer` | `32` | Soumet des expériences, consulte la bibliothèque d'assets. Ne valide pas ses propres expériences. |

!!! info "Règle d'autorisation"
    La vérification de rôle suit : `CurrentUserType.BaseUserType <= RequiredUserType`.  
    Un Admin (1) accède donc à tout ce qui requiert CompanyOwner (2), DGSManager (4), etc.

## Sous-sections

- [Hiérarchie des rôles](roles.md) — RM-008 à RM-008c
- [Rôles personnalisés & usurpation](roles-personnalises.md) — RM-009 à RM-011
- [Statuts utilisateur](statuts.md) — RM-011b à RM-011e
- [Règles techniques](technique.md) — RM-011f à RM-011n
