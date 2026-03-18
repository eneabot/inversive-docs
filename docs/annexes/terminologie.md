# Table de correspondance terminologique

## Rôles utilisateurs

| Terme métier | Terme technique | `BaseUserType` | Description |
|---|---|---|---|
| Super Admin | `Admin` | `1` | Employé Inversive, accès global |
| Administrateur | `CompanyOwner` | `2` | Client avec accès complet à sa company |
| Gestionnaire d'organisation | `DeviceGroupSetManager` | `4` | OFA Manager, gère une organisation |
| Gestionnaire d'établissement | `DeviceGroupManager` | `8` | CFA Manager, gère un établissement |
| Apprenant | `User` | `16` | Utilisateur final |
| Producteur de contenu | `Producer` | `32` | Créateur de contenu externe |

## Entités organisationnelles

| Terme métier | Terme technique | Description |
|---|---|---|
| Entreprise | `Company` | Entité cliente |
| Organisation | `DeviceGroupSet` | Niveau intermédiaire optionnel |
| Établissement | `DeviceGroup` | Unité de base de gestion |

## Appareils

| Terme métier | Terme technique | `DeviceType` | Description |
|---|---|---|---|
| Casque autonome | `DeviceStandalone` | `Standalone (3)` | VR Headset sans fil |
| Borne | `Device` (Terminal) | `Terminal (0)` | Écran ou borne interactive |

## Accès & Authentification

| Terme métier | Terme technique | Type | Description |
|---|---|---|---|
| Invité | `Guest` | `AuthenticatedUserType` | Accès via invitation d'expérience |
| Salle virtuelle / Salon | `CustomLobby` | — | Environnement 3D du casque |

## Types d'appareils (SDK)

| `DeviceTypeEnum` | Valeur | Description |
|---|---|---|
| `Terminal` | `0` | Borne/écran |
| `Personal` | `1` | Usage personnel |
| `WallScreen` | `2` | Mur interactif |
| `Standalone` | `3` | Casque autonome |
| `WebBrowser` | `4` | Navigateur web |

## Modèles de casques (SDK)

| `DeviceStandaloneTypeEnum` | Valeur |
|---|---|
| `None` | `0` |
| `Meta_Quest_2` | `1` |
| `Pico_4_Enterprise` | `2` |
| `Meta_Quest_3` | `3` |
