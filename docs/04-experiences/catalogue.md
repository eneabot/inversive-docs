# Catalogue & Affichage `[v4.0]`

## RM-029x — Filtrage selon le rôle (fonction SQL)

La liste des expériences pour le catalogue web est générée par la fonction PostgreSQL `get_front_experiences(@userId)` :

| Rôle | Expériences retournées |
|------|------------------------|
| **Admin** | Toutes les expériences |
| **Producer** | Uniquement celles dont `CreatorMail` correspond à son email |
| **CompanyOwner** | Toutes celles de sa `CompanyId` |
| **DGSManager** | Celles disponibles dans son `ExperienceDeviceGroupSet` |
| **DGManager/User** | Celles disponibles dans son `ExperienceDeviceGroup` |
| Tous | + partages explicites via `ExperienceShare` |

---

## RM-029y — IsPublished obligatoire selon le contexte

| Contexte | Règle |
|----------|-------|
| **Catalog** (web) | `IsPublished = true` obligatoire |
| **Standalone** (casque) | `IsPublished = true` obligatoire |
| **Manage** (backoffice) | Toutes les expériences non archivées, quel que soit le statut |

---

## RM-029z — ExperienceActivationState : entité d'audit distincte

`ExperienceActivationState` enregistre l'**historique des activations** (auteur, date) à des fins d'audit.  
La disponibilité opérationnelle reste portée par `ExperienceDeviceGroup` / `ExperienceDeviceGroupSet`.  
Ces deux entités sont **distinctes**.

---

## RM-029aa — Champ `Order` : réservé

Le champ `Order` dans `ExperienceDeviceGroup` et `ExperienceDeviceGroupSet` est persisté mais **n'intervient dans aucune logique de tri** côté API.  
Le tri API est effectué dynamiquement via `OrderByDynamic` sur d'autres critères.  
Réservé pour usage futur.

---

## RM-029ab — Champ `Highlighted` : non implémenté

Le champ `Highlighted` dans `ExperienceDeviceGroup` est persisté mais **n'est utilisé dans aucune logique** de filtrage, tri ou affichage.  
Réservé pour une future fonctionnalité de mise en avant.

---

## RM-029ac — HeadsetCenter : pas de pagination

Les requêtes émises par un utilisateur authentifié en tant que **HeadsetCenter** reçoivent l'intégralité des expériences **sans pagination**.  
Pour tous les autres contextes, la pagination standard (`Skip`/`Take`) s'applique.

---

## RM-029ad — Tri dans le lobby

L'utilisateur peut trier les expériences selon 3 modes (appliqué côté client Unity) :

| Mode | Description |
|------|-------------|
| **MostRecent** | Date `lastUpdate` DESC |
| **AZ** | Titre alphabétique croissant |
| **ZA** | Titre décroissant |

---

## RM-029ae — `IsFavorite` : toujours false

Le champ `IsFavorite` retourné par `get_front_experiences` est **hardcodé à `false`**.  
La fonctionnalité de gestion des favoris n'est pas implémentée dans la version actuelle.

---

## RM-029af — Compteurs calculés en SQL

| Champ | Description |
|-------|-------------|
| `TotalLaunchCount` | Nombre total de sessions pour l'expérience |
| `UserLaunchCount` | Nombre de sessions de l'utilisateur courant |

Calculés directement dans `get_front_experiences`, évitant des requêtes applicatives supplémentaires.

---

## RM-029ag — `IsPlaying` : session active

`IsPlaying = true` si l'utilisateur courant possède une `ExperienceSession` pour cette expérience dont `EndDate IS NULL` (session non terminée).

---

## RM-029ah — Recherche : comparaison approximative (fuzzy)

La recherche d'expériences par titre utilise une **comparaison approximative (fuzzy)** via `SearchHelper` dans la **langue préférée de l'utilisateur**.  
Limite d'export : **1 000 000** lignes.
