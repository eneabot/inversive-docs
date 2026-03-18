# Règles Métier — Plateforme Inversive

!!! info "Version actuelle"
    **v4.0** · Mars 2026 · **320+ règles** · Portée : `inversive.web.api` · `inversive.bll` · `inversive.data` · `inversive.common` · `headset-center` · `immersive-catalog` · `inversive_service` · `lobby-core`

Ce site est la **source de vérité** pour toutes les règles métier de la plateforme Inversive.  
Il est destiné aux développeurs, product owners, QA et toute personne qui travaille sur le produit.

---

## Comment lire ce document

Chaque règle est identifiée par un code **RM-XXX** (Rule Métier).  
Les suffixes indiquent la version d'introduction :

| Mention | Signification |
|---------|---------------|
| _(aucune)_ | Règle du Data Book v2.0 (historique) |
| `[v3.0]` | Extraite du code source en v3.0 |
| `[v4.0]` | Ajoutée ou précisée en v4.0 |

---

## Sections principales

<div class="grid cards" markdown>

-   :material-account-key: **[Authentification & Comptes](01-comptes/index.md)**  
    Activation, connexion, JWT, SSO, LTI

-   :material-account-group: **[Utilisateurs & Rôles](02-utilisateurs/index.md)**  
    6 rôles, droits, usurpation, statuts

-   :material-office-building: **[Structure organisationnelle](03-structure/index.md)**  
    Entreprise > Organisation > Établissement

-   :material-virtual-reality: **[Expériences](04-experiences/index.md)**  
    Création, validation, disponibilité, invitations

-   :material-rocket-launch: **[Lancement d'expériences](05-lancement/index.md)**  
    Catalogue web, Immersive Catalog

-   :material-headset: **[Casques autonomes](06-casques/index.md)**  
    Configuration, profils, flotte, télémétrie

-   :material-chart-bar: **[Statistiques](07-statistiques/index.md)**  
    Périmètre, filtres, export

-   :material-package-variant: **[Bibliothèque d'assets](08-assets/index.md)**  
    Statuts, formats, import groupé

-   :material-book-open-variant: **[SDK Inversive](10-sdk/index.md)**  
    Chapitrage, sessions, scoring, déploiement

-   :material-application: **[Applications & Composants](17-headset-center/index.md)**  
    Headset Center, Immersive Catalog, Service Android, Lobby, API Standalone

</div>

---

## Correspondance terminologique rapide

| Terme métier | Terme technique | Type énuméré |
|---|---|---|
| Super Admin | `Admin` | `BaseUserType = 1` |
| Administrateur | `CompanyOwner` | `BaseUserType = 2` |
| Gestionnaire d'organisation | `DeviceGroupSetManager` | `BaseUserType = 4` |
| Gestionnaire d'établissement | `DeviceGroupManager` | `BaseUserType = 8` |
| Apprenant | `User` | `BaseUserType = 16` |
| Producteur de contenu | `Producer` | `BaseUserType = 32` |
| Organisation | `DeviceGroupSet` | — |
| Établissement | `DeviceGroup` | — |
| Casque autonome | `DeviceStandalone` | — |

→ [Table complète](annexes/terminologie.md)
