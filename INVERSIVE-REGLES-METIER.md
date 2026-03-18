# Inversive Business Rules — Document Collaboratif

> **Version :** v4.0 · Mars 2026  
> **Statut :** Document vivant — modifiable par toute équipe  
>
> **Comment contribuer :**
> - Modifie directement et note [modifié par Prénom, date]
> - Marque une règle incertaine avec [À VÉRIFIER]
> - Marque une règle obsolète avec [OBSOLÈTE]
> - Ajoute un commentaire sous la règle pour ouvrir un débat

---


# Connexion

## RM-005 — SSO disponible sur web et casques

La connexion **Single Sign On (SSO)** via Keycloak est disponible :

- Sur le **site web**
- Dans les **casques autonomes**

---

## RM-006 — Connexion dans un casque par code

Pour éviter de saisir email + mot de passe dans un casque :

1. L'utilisateur génère un **code** depuis l'interface du casque
2. Il saisit ce code sur la plateforme web (menu **« Se connecter à un appareil »**)

---

## RM-007 — Connexion invité par email d'invitation

Un testeur invité peut se connecter sur un casque en saisissant l'adresse email sur laquelle il a reçu l'invitation.  
**Aucun compte plateforme n'est requis.**

---

## RM-007b — Connexion invité sur catalogue PC par code

Sur **Immersive Catalog** (catalogue PC) :

1. L'invité clique sur « Se connecter en tant qu'invité »
2. Il saisit le **code** présent dans l'email d'invitation
3. Plusieurs codes peuvent être saisis simultanément via le bouton **« + »**

---


# Création & Activation de compte

## RM-001 — Activation par email obligatoire

Tout nouveau compte doit être activé via un lien envoyé par email.  
L'utilisateur **ne peut pas se connecter** avant d'avoir cliqué sur ce lien.

---

## RM-002 — Politique de mot de passe

Le mot de passe doit comporter **au minimum 8 caractères** incluant :

- au moins une **majuscule**
- au moins une **minuscule**
- au moins un **chiffre**
- au moins un **caractère spécial** parmi : `` $@&()[]{}=#,-!?+/£€% ``

!!! warning "Note de conformité — CONF-001"
    Cette validation est appliquée par **Keycloak** (Identity Provider), pas au niveau de l'API.  
    L'API se contente de marquer le champ `Password` comme `[Required]`.  
    → Vérifier la configuration Keycloak pour s'assurer qu'elle correspond à ces règles.

---

## RM-003 — Activation différée possible

La date d'envoi du mail d'activation peut être **programmée à une date future**.  
L'utilisateur ne peut pas activer son compte avant réception de ce mail.

---

## RM-004 — Import en masse limité aux apprenants

Seuls les utilisateurs de type **Apprenant** peuvent être importés en masse via fichier Excel (`.xlsx`).  
Les autres rôles doivent être créés individuellement.

---

## RM-004b — Rapport d'erreur à l'import en masse

Lors d'un import en masse :

- Si des utilisateurs n'ont pas pu être créés → un **encart orange** liste les raisons d'échec
- Les utilisateurs valides sont **tout de même importés** (pas de rollback total)

---

## RM-004c — Utilisateurs importés avec date future : statut non activé

Si une date d'activation future est choisie lors de l'import en masse :

- Les utilisateurs sont ajoutés à la liste
- Ils restent **non activés** jusqu'à la date programmée ou une action manuelle

---


# Authentification & Comptes

Cette section couvre tout ce qui concerne la création de comptes, l'activation, la connexion et les mécanismes d'authentification techniques.

## Sous-sections

- [Création & Activation](creation-activation.md) — RM-001 à RM-004c
- [Connexion](connexion.md) — RM-005 à RM-007b
- [Règles techniques](technique.md) — RM-007c à RM-007h

---


# Règles techniques d'authentification `[v3.0]`

## RM-007c — Durée de vie d'un token JWT = 480 minutes

Les tokens JWT ont une validité de **480 minutes (8 heures)**.

```
Source : inversive.bll/Inversive.BLL/Models/JWT/JwtHelper.cs:14
```

---

## RM-007d — Ticket LTI expire après 5 minutes

Un ticket de lancement LTI est valide **5 minutes** après sa génération.  
Stocké en cache Redis avec le préfixe : `LtiLaunchTicket:`

```
Source : inversive.bll/Inversive.BLL/Services/LtiService.cs:33
```

---

## RM-007e — Authentification appareil : Serial ET MAC requis

Pour s'authentifier, un appareil doit fournir **simultanément** :

- `DeviceSerial` (numéro de série) — en en-tête HTTP
- `MacAddress` (adresse MAC) — en en-tête HTTP

Les deux doivent correspondre exactement à un appareil **non archivé**.  
La vérification remonte jusqu'à **3 niveaux** : `Device → DeviceGroup → DeviceGroupSet`.

```
Source : inversive.web.api/Inversive.Web.Api/Auth/AuthorizeDeviceAttribute.cs:53
```

---

## RM-007f — Module Standalone absent → HTTP 423 (pas 403)

Lorsqu'un endpoint réservé aux casques autonomes est appelé par une company sans le module `Standalone` activé :

- Réponse : **HTTP 423 (Locked)**
- ≠ HTTP 403 (Forbidden) — le 423 signifie que la fonctionnalité est verrouillée pour raisons **contractuelles**

```
Source : inversive.web.api/Inversive.Web.Api/Auth/AuthorizeStandalone.cs:101
```

---

## RM-007g — Premier login : enregistrement de la date d'activation

Lors de la **première connexion réussie** d'un utilisateur :

- Si `ActivationDate == null` → il est automatiquement renseigné avec `DateTime.UtcNow`

```
Source : inversive.web.api/Inversive.Web.Api/Auth/AuthorizeUserRightsAttribute.cs:90
```

---

## RM-007h — Réinitialisation de mot de passe : confirmation obligatoire

Lors d'une réinitialisation de mot de passe, `ConfirmPassword` doit correspondre exactement à `NewPassword`.  
Validation via `[Compare("NewPassword")]` côté modèle.

```
Source : inversive.common.web/Inversive.Common.Web/Account/ResetPasswordModel.cs
```

---


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

---


# Rôles personnalisés & Usurpation

## RM-009 — Impossible d'accorder des droits supérieurs aux siens

Un utilisateur ne peut pas créer de rôle personnalisé avec des droits supérieurs aux siens.  
Un gestionnaire ne peut pas accorder des droits d'administrateur.

---

## RM-010 — Usurpation de rôle limitée vers le bas

La fonctionnalité d'**usurpation de rôle** (disponible pour Administrateurs et Gestionnaires) ne permet pas d'usurper un rôle supérieur au sien dans la matrice des droits.

---

## RM-011 — Actions d'usurpation sont effectives

!!! warning "Attention"
    Toutes les actions effectuées durant une session d'usurpation de rôle sont **réelles et prises en compte** sur la plateforme (modification de disponibilité, soumission d'expérience, etc.).

---


# Hiérarchie des rôles

## RM-008 — Héritage des droits vers le haut

Les droits accordés à un rôle doivent **obligatoirement être accordés aux rôles de niveau supérieur**.  
Ce qu'un gestionnaire peut faire doit pouvoir être fait par un administrateur.

---

## RM-008b — Périmètre du Gestionnaire d'organisation

Le gestionnaire d'organisation peut :

- ✅ Gérer les expériences qu'**il a créées**, au sein de son organisation
- ✅ Modifier la disponibilité des expériences dans les établissements de son organisation
- ❌ Créer des utilisateurs
- ❌ Gérer des expériences créées par d'autres

---

## RM-008c — Périmètre du Gestionnaire d'établissement

Le gestionnaire d'établissement peut :

- ✅ Gérer les expériences qu'**il a créées** au sein de son établissement
- ✅ Gérer les utilisateurs dans son unique établissement
- ❌ Intervenir dans d'autres établissements

---


# Statuts utilisateur

Un utilisateur peut avoir l'un des 5 statuts suivants :

| Statut | Description |
|--------|-------------|
| **Activé** | Compte actif, l'utilisateur peut se connecter |
| **Non activé** | Mail d'activation reçu mais mot de passe non encore défini |
| **Prévu le [date]** | Mail d'activation programmé, compte inaccessible jusqu'à réception |
| **Archivé** | Compte désactivé, réactivable depuis la section Archives |
| **Erreur** | Une erreur est survenue lors de la création |

---

## RM-011b — Envoi d'email d'activation groupé

Il est possible d'envoyer un mail d'activation à **tous les comptes « non activés »** d'une sélection multiple en une seule action.

---

## RM-011c — Archivage individuel ou programmé

Un utilisateur peut être archivé :

- **Immédiatement**
- **À une date future** programmée

L'archivage peut être appliqué à une **sélection multiple** d'utilisateurs.

---

## RM-011d — Anonymisation d'utilisateur

La plateforme propose une action **« Anonymiser »** applicable individuellement ou sur une sélection multiple.  
Cette action est **distincte de l'archivage**.

---

## RM-011e — Affectation de rôle en masse

Il est possible d'affecter un même rôle à tous les utilisateurs d'une sélection multiple en une seule action.

---


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

---


# Archivage

Voir la section [Archivage](index.md#archivage) de la page principale.

---


# Hiérarchie organisationnelle

Voir [Structure organisationnelle](index.md) pour le contenu complet.

---


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

---


# Formats & Binaires

## RM-026 — Un binaire par type d'appareil

Pour une même expérience disponible sur plusieurs types d'appareils, un **binaire distinct** doit être fourni pour chaque type.

| Type d'appareil | Format binaire |
|-----------------|----------------|
| Casque sans fil | `.apk` |
| Navigateur | WebGL (`.zip`) |
| PC filaire (SteamVR) | `.exe` |

---

## RM-027 — Persistance de session cross-appareils via SDK

Une expérience intégrant le SDK Inversive peut :

- **Enregistrer la progression** d'un utilisateur
- Lui permettre de **reprendre sur un autre appareil** exactement là où il s'était arrêté

La session reste ouverte jusqu'à l'appel de la fonction `End()`.

---

## RM-028 — Point d'entrée obligatoire pour les exécutables multi-.exe

Si un binaire contient **plusieurs `.exe`**, l'auteur doit obligatoirement en sélectionner un comme **point d'entrée** lors de la finalisation de l'expérience.

---

## RM-028b — Modèle de casque obligatoire pour les expériences sans fil

Lors de l'ajout d'un binaire de type « Casque sans fil » :

- Le **modèle de casque cible** doit être sélectionné
- Un même contenu peut être décliné pour **plusieurs modèles** de casques

---

## RM-029 — Suppression = envoi en archives

Une expérience supprimée n'est **pas définitivement effacée** : elle est déplacée dans la section **Archives** d'où elle peut être désarchivée.

---

## RM-029b — Modification d'une expérience

La modification d'une expérience ouvre un formulaire **identique à la création**.  
Toute modification doit être validée via **« Enregistrer les modifications »**.

---


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

---


# Création d'une expérience

## RM-015c — Processus de création en 7 étapes

| Étape | Description |
|-------|-------------|
| 1 | Ajout des binaires |
| 2 | Ajout des médias |
| 3 | Choix des langues |
| 4 | Choix de la catégorie *(facultatif)* |
| 5 | Ajout de chapitres SDK *(facultatif)* / Invitation de testeurs |
| 6 | Finalisation du point d'entrée |
| 7 | Gestion de l'activation |

---

## RM-015d — Types d'expériences disponibles

| Type | Détail |
|------|--------|
| Casque filaire | SteamVR |
| Navigateur | WebGL |
| Vidéo 360 | — |
| Casque sans fil | APK — nécessite de sélectionner le modèle de casque |
| Mur interactif | — |
| Réalité Augmentée | APK Android |
| Standard | .exe Windows |
| Podcast | Audio |
| Vidéo | — |
| BIM | — |
| Application externe | — |
| Uptale | Conditionné par `IsUptaleEnabled` |

!!! info
    Les types activés dépendent de la **configuration contractuelle** du client.  
    → Voir [RM-014e — Types d'expériences filtrés par modules](../03-structure/index.md)

---

## RM-015e — Médias d'une expérience

À la création, il est possible d'associer :

- Une **icône**
- Une **image de fond** (image principale)
- Un **trailer** au format `.mp4`
- Jusqu'à **4 captures d'écran**

Ces médias sont modifiables à tout moment.

---

## RM-015f — Titre et description multilingues obligatoires

Pour chaque langue configurée, les éléments suivants doivent être renseignés :

- **Titre**
- **Description**
- **Contrôles** de l'expérience

Une expérience peut être décrite dans plusieurs langues simultanément.

---


# Disponibilité des expériences

## RM-022 — 3 niveaux de déploiement obligatoires

Pour qu'une expérience soit accessible aux utilisateurs finaux, **3 étapes successives** sont requises :

```
1. Activation par l'administrateur au niveau Organisation/Autre structure
        ↓
2. Mise à disposition au niveau Établissement par le gestionnaire
        ↓
3. Accès effectif aux utilisateurs et casques de l'établissement
```

---

## RM-023 — Désactivation au niveau Organisation : cascade sur les établissements

Si un contenu est retiré d'une Organisation, il est **automatiquement et immédiatement** retiré de **tous les Établissements** qui composent cette Organisation.

---

## RM-024 — Expérience désactivée = visible uniquement par les admins

Une expérience non activée dans une Organisation est uniquement visible par les **administrateurs**.  
Les gestionnaires et apprenants n'y ont pas accès.

---

## RM-025 — Expériences sur casque filtrées par compatibilité + établissement

Dans un profil de casque, les expériences disponibles à la sélection sont celles qui sont :

- ✅ **Compatible** avec le modèle de casque
- ✅ **Disponible** dans l'établissement associé au profil

Les deux conditions doivent être réunies simultanément.

---

## RM-025b — Disponibilité au niveau Organisation ne suffit pas

!!! warning
    Activer une expérience dans une Organisation **ne la rend pas directement accessible** aux utilisateurs finaux.  
    Elle doit encore être mise à disposition au niveau de chaque **Établissement** concerné.

---


# Expériences

## Statuts d'une expérience

| Statut | Description |
|--------|-------------|
| **Actif** | Expérience validée et fonctionnelle |
| **Inactive** | Validée mais non déployée dans aucun établissement |
| **À valider** | En phase de tests |
| **Refusée** | Refusée par l'administrateur, doit être corrigée |

## Sous-sections

- [Création](creation.md) — RM-015c à RM-015f
- [Validation & Tests](validation.md) — RM-016 à RM-021e
- [Disponibilité](disponibilite.md) — RM-022 à RM-025b
- [Formats & Binaires](binaires.md) — RM-026 à RM-029b
- [Invitations](invitations.md) — RM-029c à RM-029p
- [Catalogue & Affichage](catalogue.md) — RM-029x à RM-029ah
- [Règles techniques](technique.md) — RM-029q à RM-029w

---


# Invitations & Accès casque `[v4.0]`

## Filtre de base des expériences (endpoint `/experience/init`)

### RM-029c — Conditions obligatoires pour servir une expérience au casque

L'endpoint `/experience/init` (authentification Device) ne retourne que les expériences satisfaisant **simultanément** :

- `IsCreationDone = true`
- `IsPublished = true`
- `Archived = false`

```
Source : inversive.web.api/Controllers/ExperienceController.cs:406
```

---

### RM-029d — Mode DGS : double condition de disponibilité

Lorsqu'une company utilise le module `DeviceGroupSet`, une expérience n'est disponible sur un casque que si `Available = true` à la fois :

- Au niveau **`ExperienceDeviceGroupSet`** (organisation)
- Au niveau **`ExperienceDeviceGroup`** (établissement)

La disponibilité à un seul niveau est **insuffisante**.

```
Source : inversive.web.api/Controllers/ExperienceController.cs:350
```

---

### RM-029e — Expériences invitées : `!IsPublished` obligatoire

Les expériences accessibles via invitation (`ExperienceInvitation` ou `ExperienceInvitationDeviceStandalone`) sont **exclusivement** des expériences **non publiées** (`IsPublished = false`).  
Une expérience déjà publiée/active ne passe pas par le système d'invitation.

```
Source : inversive.web.api/Controllers/ExperienceController.cs:391
```

---

### RM-029f — Package APK absent → état `removed` automatique

Si une expérience est marquée `isInstalled = true` dans le cache local mais que le package n'est plus présent sur l'appareil → elle est automatiquement repassée à l'état **`removed`**.  
Inversement, si un package est installé sans être marqué → il est marqué **`installed`**.

```
Source : inversive_service/.../RefreshExperiencesUseCase.kt:60
```

---

### RM-029g — Expérience de test : lancement bloqué sans invitation valide

Dans le lobby, une expérience `isTesting = true` ne peut être lancée que si `allowedByInvitation = true`.  
Ce flag n'est activé que si la vérification d'invitation (par email ou token) a réussi.

```
Source : lobby-core/.../UI/ExperienceButtons/ExperienceThumbnailButton.cs:307
```

---

## ExperienceInvitation — Règles détaillées

### RM-029h — Unicité (email + expérience) avec upsert

Il ne peut exister qu'une seule invitation par combinaison `(email, experienceId)`.  
Si une invitation pour ce couple existe déjà → elle est **mise à jour** plutôt qu'une nouvelle entrée créée.  
Un email de notification est renvoyé à chaque fois.

```
Source : inversive.web.api/Controllers/ExperienceInvitationController.cs:727
```

---

### RM-029i — Durée par défaut = 15 jours

- Durée par défaut : **15 jours**
- Expiration calculée : `DateTime.UtcNow + TimeSpan.FromDays(Duration)`
- La modification de durée recalcule l'expiration depuis `UtcNow` (pas depuis la création initiale)

```
Source : inversive.web.api/Controllers/ExperienceInvitationController.cs:729, 914
```

---

### RM-029j — Création batch avec déduplication

L'endpoint de création accepte une liste d'invitations.  
Les doublons d'email sont éliminés par `.DistinctBy(x => x.Email)` avant traitement.

```
Source : inversive.web.api/Controllers/ExperienceInvitationController.cs:713
```

---

### RM-029k — UserId renseigné si l'invité est un utilisateur existant

Lors de la création d'une invitation, la plateforme recherche un utilisateur existant avec le même email.  
Si trouvé : `UserId` + `FirstName` / `LastName` sont automatiquement renseignés.

```
Source : inversive.web.api/Controllers/ExperienceInvitationController.cs:717
```

---

### RM-029l — Les invitations ne sont pas supprimées automatiquement

!!! info
    Aucun job de nettoyage automatique. Les invitations expirées **persistent indéfiniment** en base.  
    Filtrage `Expiration >= DateTime.Today` appliqué à la lecture.  
    Suppression manuelle via `remove-invitation/{id}` (droit Delete requis).

```
Source : inversive.web.api/Controllers/ExperienceInvitationController.cs:867
```

---

### RM-029m — Invitations Device-Standalone : validité indéfinie

Les invitations `ExperienceInvitationDeviceStandalone` (liant une expérience à un casque spécifique) :

- **Pas de champs** `Duration`, `Creation` ou `Expiration`
- Valides **indéfiniment** jusqu'à suppression explicite
- Endpoint de nettoyage : `clean-device-standalone-invitations/{deviceId}`

---

### RM-029n — Authentification invité via header `Invitation`

Pour accéder en tant qu'invité (`Guest`) :  
→ Transmettre l'ID de l'invitation dans le header HTTP `Invitation: {invitationId}`

Le `GuestAuthHandler` intercepte ce header, vérifie l'invitation, puis crée un `ClaimsPrincipal` de type Guest avec `NameIdentifier = invitationId`.

```
Source : inversive.web.api/Auth/GuestAuthHandler.cs:33
```

---

### RM-029o — Un invité peut soumettre un feedback complet

Les endpoints `create-notation` et `upload-notation-screenshot` acceptent une authentification **Guest**.  
Un testeur sans compte plateforme peut donc soumettre :

- Like/dislike
- Commentaire
- Jusqu'à **5 captures d'écran**

---

### RM-029p — Endpoint de vérification d'expiration en batch (anonyme)

`POST check-invites-expiration` (AllowAnonymous) :

- Accepte une liste d'IDs d'invitations
- Retourne pour chacune : `(id, isValid)` — valide si `Expiration >= DateTime.Today`
- **Accessible sans authentification**

---


# Règles techniques des expériences `[v3.0]`

## RM-029q — Taille maximale d'un binaire = 10 Go

La taille maximale acceptée pour un fichier binaire lors de l'upload est de **10 gigaoctets** (`10 × 1 024 × 1 024 × 1 024` bytes).

```
Source : inversive.web.api/Inversive.Web.Api/Controllers/ExperienceCreationController.cs:23
```

---

## RM-029r — Formats de binaires externes autorisés

Les fichiers binaires externes (sous-titres, ambiances sonores) sont limités aux extensions :

`.amb` · `.mp3` · `.wav` · `.flac` · `.srt`

```
Source : inversive.web.api/Inversive.Web.Api/Controllers/ExperienceCreationController.cs:21
```

---

## RM-029s — Fallback automatique de langue

Lorsque le titre ou la description n'est pas disponible dans la langue demandée, le fallback est :

1. Langue demandée
2. Français (`LangId = 1`)
3. Première langue disponible

```
Source : inversive.bll/Inversive.BLL/GlobalRules/ExperienceHelper.cs:8
```

---

## RM-029t — Sessions développeur marquées séparément

Les sessions de test lancées depuis Unity en mode **PlayMode** (via le SDK) sont marquées `IsDeveloper = true` dans `ExperienceSession`.  
Ces sessions sont **exclues des statistiques** de production.

---

## RM-029u — Partage d'expériences entre companies

Une expérience peut être partagée avec d'autres companies via la relation `ExperienceShare`.

---

## RM-029v — Types de binaires externes

| Valeur | Type |
|--------|------|
| `0` | Audio |
| `1` | Subtitle |

---

## RM-029w — Angles de projection pour la vidéo 360°

Les 6 angles de projection supportés :

`180` · `180LR` (Left-Right) · `180TB` (Top-Bottom) · `360` · `360LR` · `360TB`

Le format de projection doit être défini lors de la configuration du binaire.

---


# Validation & Tests

## RM-016 — Deux voies de finalisation

Lors de la création, l'auteur choisit entre :

| Voie | Statut résultant |
|------|-----------------|
| **Valider sans tester** | → **Actif** directement |
| **Inviter des testeurs** | → **À valider** (phase de test) |

---

## RM-017 — Invitation de testeurs sans compte plateforme

Il n'est **pas nécessaire** d'avoir un compte sur la plateforme pour tester une expérience en tant qu'invité.  
L'invitation se fait par email avec un lien d'accès valable **15 jours par défaut**.

!!! note "Note de conformité — CONF-002"
    Le code vérifie `Expiration < DateTime.Today`, ce qui inclut le **jour d'expiration jusqu'en fin de journée** (23:59:59).

---

## RM-018 — Durée de validité d'une invitation éditable

La période de validité (15 jours par défaut) peut être éditée à tout moment par l'administrateur.

---

## RM-019 — Retour de test possible après validation

Même après validation (statut **Actif**), il reste possible d'inviter des utilisateurs à tester l'expérience.

---

## RM-020 — Cycle refus / correction / re-validation

1. L'expérience est **refusée** → le producteur reçoit un email de notification
2. Le producteur **corrige** les binaires → statut repasse à **À valider**
3. Nouveau cycle de validation

Les informations de refus sont visibles dans l'onglet **« Suivi des refus »**.

---

## RM-021 — Retour de test : commentaire + jusqu'à 5 images

Un testeur peut approuver ou refuser une expérience en joignant :

- Un **commentaire**
- Jusqu'à **5 images**

Un administrateur refusant une expérience peut de même justifier sa décision (commentaire + 5 images).

---

## RM-021b — Test sur casque autonome : casque doit appartenir à la flotte

Pour ajouter une expérience en test sur un casque autonome, ce casque doit **impérativement appartenir à la flotte** du compte administrateur.  
Sinon, il n'apparaît pas dans la liste de sélection.

---

## RM-021c — Expérience en test sur casque : affichage en noir et blanc

Sur un casque, une expérience en cours de test s'affiche **en noir et blanc**.  
Pour la lancer, l'utilisateur doit :

- Être connecté avec un compte ayant accès aux expériences de test, **ou**
- Avoir été invité

---

## RM-021d — Commentaires depuis l'expérience elle-même

Il est possible d'effectuer des commentaires de test **depuis l'intérieur de l'expérience**, à condition que :

- L'expérience soit développée pour **Pico Enterprise**
- Elle intègre le **SDK Inversive**

---

## RM-021e — Retours classés par ordre alphabétique puis chronologique

Dans l'interface de consultation des retours de test, le tri est :

1. Ordre alphabétique du testeur
2. Ordre chronologique du plus récent au plus ancien
3. Par type de média

---


# Lancement d'expériences

## Catalogue web

### RM-030a — Carrousels par format

Le catalogue web organise les expériences en **carrousels par format** :

VR filaire · Web (WebGL) · Vidéo 360 · Casques autonomes · Écrans muraux · Réalité Augmentée · Standard · Applications externes · Podcast · Vidéo · BIM

---

### RM-030b — Expériences non WebGL : téléchargement du logiciel requis

Si une expérience n'est pas de type WebGL ou Vidéo 360, la plateforme propose le téléchargement de la suite logicielle nécessaire :

- **Immersive Catalog** — pour les expériences VR filaires et standard
- **Headset Center** — pour les casques autonomes

---

## Catalogue PC (Immersive Catalog)

### RM-030c — Identifiants identiques au site web

Les identifiants de connexion au catalogue PC (**Immersive Catalog Launcher**) sont **identiques** à ceux du site web Inversive.

---

### RM-030d — Sélection des expériences à la première connexion

À la première connexion sur Immersive Catalog, l'utilisateur peut choisir les expériences à télécharger.  
Cette étape peut être **ignorée** ; les expériences restent téléchargeables depuis le catalogue à tout moment.

---

### RM-030e — Déconnexion via l'icône système

Pour se déconnecter ou changer d'utilisateur sur Immersive Catalog :

1. Cliquer sur l'icône du logiciel dans la **barre des tâches Windows**
2. Ouvrir le **configurateur**

---


# Actions sur casques

## RM-036 — Mode Kiosk, Éteindre et Redémarrer : Pico uniquement

Les actions suivantes sont disponibles **uniquement pour les casques Pico Enterprise** :

- Activer/Désactiver le **Mode Kiosk**
- **Éteindre** un casque à distance
- **Redémarrer** un casque à distance

!!! warning "Note de conformité — CONF-003"
    Dans le modèle de données, `KioskMode` est présent sur **tous** les `DeviceStandalone` sans filtrage par constructeur.  
    La restriction Pico est appliquée au niveau de l'**interface utilisateur**, pas du modèle de données.

!!! info
    L'activation/désactivation du Mode Kiosk entraîne un **redémarrage** du casque.

---

## RM-037 — Actions à distance : même réseau Wi-Fi requis

Pour que les actions à distance soient disponibles (casting, lancement d'expérience, mode kiosk…), le **Headset Center et les casques doivent être sur le même réseau Wi-Fi**.

---

## RM-038 — Mot de passe kiosk

Le mot de passe permettant de désactiver ou réactiver le Mode Kiosk est connu **uniquement des utilisateurs ayant accès aux profils de casques**.

---

## RM-038b — Casting = vue en direct

L'action **Casting** (mise en miroir) permet de voir en temps réel ce que l'utilisateur voit dans le casque.  
Le **multicasting** permet de visualiser plusieurs flux de casques simultanément.

---

## RM-038c — Actions groupées : bouton actif dès 1 casque sélectionné

Le bouton « Actions » sur une sélection multiple de casques devient **actif dès qu'au moins un casque est sélectionné**.  
Les actions s'affichent pour tous les casques sélectionnés, même si certaines ne sont effectives que sur certains modèles.

---

## RM-038d — Export des données de la flotte

Il est possible d'**exporter les données de la flotte** de casques depuis l'interface web (page Appareils).

---


# Configuration initiale des casques

## RM-030 — Configuration via PC obligatoire

L'ajout d'un casque autonome à la flotte nécessite de le connecter **physiquement (câble USB)** à un PC sur lequel le **Headset Center** est installé.  
C'est une étape **obligatoire**.

---

## RM-031 — Rattachement à un établissement lors de la configuration

Lors de la configuration :

1. Sélectionner le **modèle de casque**
2. Sélectionner l'**établissement** *(si l'établissement appartient à une Organisation, sélectionner d'abord l'Organisation)*

---

## RM-031b — Prérequis Meta Quest : mode développeur activé

Pour configurer un casque **Meta Quest**, il doit :

- Être **allumé**
- Avoir passé la **configuration initiale**
- Être en **mode développeur**

!!! tip
    Si l'écran d'autorisation Debug USB n'apparaît pas, appuyer sur le **bouton Meta** pour l'afficher.

---

## RM-031c — Redéploiement de la même configuration sur un nouveau casque

Une fois la configuration terminée, il est possible de **redéployer la même configuration** sur un nouveau casque du même modèle directement depuis l'écran de finalisation.

---

## RM-032 — Mise à jour autonome si connecté à internet

Une fois configuré, le casque se **maintient à jour de façon autonome** dès lors qu'il est connecté à internet.  
Aucune intervention manuelle n'est requise pour les mises à jour logicielles.

---


# Casques autonomes

Cette section couvre tout ce qui concerne la gestion des casques VR autonomes : configuration initiale, profils, actions à distance, réinitialisation et règles techniques.

## Sous-sections

- [Configuration initiale](configuration.md) — RM-030 à RM-032
- [Profils de casques](profils.md) — RM-033 à RM-035b
- [Actions sur casques](actions.md) — RM-036 à RM-038d
- [Réinitialisation](reinitialisation.md) — RM-038e à RM-038g
- [Règles techniques](technique.md) — RM-038h à RM-038j

## Casques supportés

| Constructeur | Modèles |
|---|---|
| **Meta** | Quest 2, Quest 3, Quest 3S |
| **Pico** | G3, 4 Enterprise, 4 Ultra Enterprise, Neo 3 Pro, Neo 3 Pro Eye |
| **HTC** | Vive Focus Vision, Vive XR Elite |

---


# Profils de casques

## RM-033 — Modification d'un profil impacte tous les casques associés

Modifier un profil de casque (paramètres ou liste d'expériences) applique **immédiatement** les changements à **tous les casques** associés à ce profil.

---

## RM-034 — Modification individuelle désynchronise du profil

Modifier manuellement les expériences d'un casque individuel **désynchronise ce casque** de son profil.  
Il est possible de réappliquer un profil ultérieurement.

---

## RM-035 — Un groupe de casques = un seul modèle constructeur

Un groupe de casques dans le Headset Center ne peut contenir que des casques du **même modèle constructeur**.

---

## RM-035b — Réseau Wi-Fi : sensible à la casse

!!! warning
    Le réseau Wi-Fi enregistré dans un profil de casque ne sera **pas reconnu** en cas de faute de frappe.  
    La saisie du SSID doit être **exacte** (respect de la casse).

---


# Réinitialisation des casques

## RM-038e — Remise à l'état d'usine — Pico 4 Enterprise

=== "Étapes"
    1. Casque **éteint**
    2. Maintenir **Volume+** puis **Power** jusqu'à « No command »
    3. Maintenir **Power** + appuyer sur **Volume+** → affiche le menu Android
    4. Naviguer avec **Volume+/-** → sélectionner **« Factory reset »** → valider avec **Power**
    5. Sélectionner **« Reboot system now »**

---

## RM-038f — Remise à l'état d'usine — Meta Quest 2

=== "Étapes"
    1. Casque **éteint**
    2. Maintenir **Volume-** puis **Power** jusqu'à l'apparition du menu Android
    3. Naviguer avec **Volume+/-** → sélectionner **« Factory reset »** → valider avec **Power**
    4. Sélectionner **« Boot device »**

---

## RM-038g — Activation du mode développeur Meta Quest

Via l'application **Meta Horizon** sur smartphone :

**Menu → Appareils → Mode développeur**

!!! info "Prérequis"
    Le **Bluetooth**, la **localisation** et le **Wi-Fi** (même réseau que le casque) doivent être actifs sur le téléphone.

---


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

---


# Statistiques

## RM-039 — Périmètre selon le rôle

| Rôle | Périmètre visible |
|------|-------------------|
| **Producteur** | Ses propres expériences |
| **Administrateur** | Ensemble de son Entreprise |
| **Gestionnaire** | Son Organisation / Établissement |
| **Apprenant** | Ses propres lancements |

---

## RM-040 — Statistiques additionnelles pour les administrateurs

Les administrateurs peuvent consulter en plus :

- Les statistiques de **téléchargements des assets**
- L'**évolution du parc d'utilisateurs**

---

## RM-041 — Export en .xlsx

Les données statistiques filtrées peuvent être **exportées au format Excel** (`.xlsx`) depuis la page Statistiques.

---

## RM-041b — Filtres disponibles

| Filtre | Type |
|--------|------|
| Organisation | Sélection |
| Autre structure | Sélection |
| Période de date | Plage |
| Type d'expérience | Sélection |
| Statut | Sélection |
| Nom d'expérience | Texte |
| Utilisateur spécifique | Recherche |

---

## RM-041c — 3 onglets de données

| Onglet | Contenu |
|--------|---------|
| **Expériences** | Lancements, types |
| **Utilisateurs** | Activité |
| **Assets** | Téléchargements |

---


# Bibliothèque d'assets

## Statuts d'un asset

| Statut | Description |
|--------|-------------|
| **Publié** | Visible dans la bibliothèque |
| **Dans le panier** | Sélectionné par l'utilisateur |
| **En cours de demande d'accès** | En attente de validation admin |
| **Disponible au téléchargement** | Accès accordé par l'admin |

---

## RM-042 — 4 statuts pour un asset

Voir tableau ci-dessus.

---

## RM-043 — Demande d'accès validée par l'administrateur

L'accès à un asset est accordé ou refusé par l'**administrateur de l'entreprise** après réception d'une notification email.  
Le statut est mis à jour en **temps réel**.

---

## RM-044 — Formats acceptés par type d'asset

| Type | Formats acceptés |
|------|-----------------|
| **3D** | `.obj`, `.glb`, `.gltf`, `.fbx` |
| **2D** | `.jpg`, `.jpeg`, `.png` |
| **VFX** | `.psd`, `.tga`, `.tiff`, `.dpx` |
| **UnityPackage** | — |
| **Vidéo** | `.mp4` |
| **Audio** | `.mp3`, `.wav`, `.m4a` |

!!! info
    Tous les fichiers doivent être fournis dans un dossier compressé **`.zip`**.

---

## RM-045 — Jusqu'à 4 formats simultanés pour un asset 3D

Pour un même asset 3D, il est possible de téléverser **jusqu'à 4 formats simultanément** (`.obj`, `.glb`, `.gltf`, `.fbx`).

---

## RM-046 — Publication notifie les administrateurs

Lors de la publication d'un nouvel asset, **tous les administrateurs** de l'entreprise concernée reçoivent un email de notification.

---

## RM-046b — Import groupé via .zip structuré

Il est possible d'importer plusieurs assets via un **fichier .zip respectant une architecture de dossiers** précise (téléchargeable comme modèle).  
Un fichier `.json` peut compléter les métadonnées.

---

## RM-046c — Prévisualisation avant import groupé

Avant de lancer l'import, une prévisualisation récapitule :

- Nombre total d'assets
- Nombre de dossiers valides
- Nombre de dossiers en erreur

Chaque asset peut être **modifié individuellement** avant l'import.

---

## RM-046d — Reprise possible si import interrompu

Si une erreur survient durant un import groupé, il est possible de **relancer l'import uniquement pour les assets dont l'import a été interrompu**.

---

## RM-046e — Types d'assets disponibles

Modèles 3D · Images 2D · VFX · Animations · Vidéos · Textures et matériaux · Audios

---

## Règles techniques `[v3.0]`

### RM-046f — BundleAsset IsVanilla : accessible à toutes les companies

Un `BundleAsset` avec `IsVanilla = true` est accessible à **toutes les companies** sans restriction.  
Pas besoin d'association `CompanyBundleAsset`.

### RM-046g — BundleAsset non-vanilla : accès restreint

Un `BundleAsset` avec `IsVanilla = false` n'est accessible que si la company possède une association `CompanyBundleAsset` correspondante.

### RM-046h — Types de contenus AssetBundle

| Valeur | Type | Description |
|--------|------|-------------|
| `0` | Theme/Décor | Ambiances visuelles |
| `1` | AdditionalContent | Contenus additionnels |
| `2` | Scene | Scènes 3D complètes |

---


# Règles techniques des assets

Voir la section [Règles techniques](index.md) dans la page principale (bas de page).

---


# Guides d'utilisation

## RM-047 — Accès réservé aux Gestionnaires et supérieurs

Seuls les utilisateurs ayant le rôle **Administrateur ou Gestionnaire** peuvent :

- Ajouter un guide
- Commenter un guide
- Afficher un guide
- Télécharger un guide
- Supprimer un guide

---

## RM-048 — Format PDF uniquement

Les guides d'utilisation doivent être importés au format **PDF (`.pdf`)** exclusivement.

---

## RM-048b — Accès depuis la fiche produit du catalogue

Les guides sont accessibles depuis la **fiche produit de l'expérience** dans le catalogue, sous la description.  
Visibles par les utilisateurs ayant le rôle **Gestionnaire ou supérieur**.

---

## RM-048c — Commentaires sur les guides

Les administrateurs et gestionnaires peuvent ajouter des **commentaires** sur chaque guide d'utilisation.  
Les commentaires sont **affichables/masquables**.

---

## RM-048d — Documentation personnalisée dans Aide & Support

Les administrateurs peuvent ajouter de la documentation personnalisée dans la section **« Aide et Support »** :

- Ciblée par **type d'utilisateur**
- Format : **URL** ou **fichier PDF**

---


# Configuration & Déploiement `[v4.0]`

## RM-053j — SDK Production vs SDK Dev

Les deux packages Unity sont **fonctionnellement identiques**. Seule l'URL de base des appels API diffère :

| Package | Version | URL de base |
|---------|---------|------------|
| `inversive-sdk` | v1.1.2 | `https://sdk.vrcxp.com/` |
| `inversive-dev-sdk` | v1.1.6 | `https://dev-sdk.vrcxp.com/` |

```
Source : inversive-sdk/Runtime/InversiveUtilities.cs:8
         inversive-dev-sdk/Runtime/InversiveUtilities.cs:8
```

---

## RM-053k — 5 types de dispositifs supportés

Le SDK reconnaît 5 modes via `DeviceTypeEnum` :

| Valeur | Type |
|--------|------|
| `0` | Terminal |
| `1` | Personal |
| `2` | WallScreen |
| `3` | Standalone |
| `4` | WebBrowser |

Pour le type `Standalone`, un sous-type `DeviceStandaloneTypeEnum` précise le modèle :

| Valeur | Modèle |
|--------|--------|
| `0` | None |
| `1` | Meta Quest 2 |
| `2` | Pico 4 Enterprise |
| `3` | Meta Quest 3 |

```
Source : inversive-sdk/Runtime/InversiveClasses.cs:222-243
```

---


# SDK Inversive

Le SDK Inversive s'intègre dans les projets **Unity** pour gérer le chapitrage, le scoring, les sessions et le reporting.

## Prérequis

- **Unity** version 2019.1 ou supérieure
- **Compte actif** sur la plateforme (administrateur ou producteur de contenu)
- Installation via Package Manager : `https://github.com/InversivePackages/inversive-sdk`

---

## RM-049 — App ID lié à un compte unique

Un **App ID** est associé à la création d'une expérience sur un compte précis.  
Si plusieurs comptes sont disponibles, l'expérience doit être créée/publiée avec le profil qui a généré l'App ID.

---

## RM-049b — App ID : remplit automatiquement le formulaire de création

Une fois un App ID généré et le modèle poussé (Push Model), le formulaire de création se **remplit automatiquement** sur le compte correspondant.

---

## RM-049c — App ID partageable

L'App ID peut être transmis à d'autres développeurs (avec ou sans compte Inversive) pour qu'ils puissent modifier ou exploiter le chapitrage.

---

## RM-050 — Push Model écrase le précédent

L'action **« Push Model »** remplace **définitivement** le modèle précédemment poussé pour cet App ID.  
De même, un modèle chargé écrase le modèle actuel dans l'éditeur.

---

## RM-050b — Export du modèle en JSON

Le modèle de chapitrage peut être exporté en fichier JSON (bouton **« Export Model »**), placé dans `Assets/Inversive SDK`.  
Si un export précédent existe, il est **remplacé**.

---

## Scoring

### RM-051 — Score global sur 100 points

Le SDK calcule le score total sur **100 points**, somme des scores des actions.  
Ce score est ensuite converti selon le type de notation choisi :

| Type | Plage |
|------|-------|
| Standard | 0–100 |
| Décimal | 0–10 |
| Vingtième | 0–20 |
| Lettre | A+/F |

### RM-051b — Table de correspondance A+/F

| Note | Seuil |
|------|-------|
| A+ | ≥ 75 |
| A | 70–74 |
| A- | 65–69 |
| B+ | 60–64 |
| B | 55–59 |
| B- | 50–54 |
| C+ | 45–49 |
| C | 40–44 |
| C- | 35–39 |
| D+ | 30–34 |
| D | 25–29 |
| D- | 20–24 |
| F | < 20 |

### RM-052 — Win Score à définir par l'auteur

L'auteur doit définir un **seuil de score** (Win Score) à partir duquel l'expérience est considérée comme réussie.  
En dessous de ce seuil, la session est marquée comme **échouée**.

---

## Structure du modèle

### RM-052b — Hiérarchie : Modèle > Chapitres > Actions

```
Modèle
└── Chapitre 1
│   ├── Action A
│   └── Action B
└── Chapitre 2
    └── Action C
```

L'ordre des chapitres est défini explicitement.

### RM-052c — Types de valeurs d'une action

| Type | Détail |
|------|--------|
| **Boolean** | Vrai/faux, réponse unique uniquement |
| **String** | Texte, réponse unique ou multiple |
| **Float** | Virgule flottante — **séparateur virgule** (pas point) |
| **Integer** | Nombre entier |

Float et Integer acceptent aussi les types réponse Intervalle et Paliers.

### RM-052d — Types de réponses d'une action

| Type | Score |
|------|-------|
| **Unique** | Bonne réponse = 100 pts fixe |
| **Multiple** | Scores rapportés sur 100 pts |
| **Intervalle** | Calculé proportionnellement à la proximité de la valeur cible |
| **Paliers** | Intervalles avec scores associés fixes |

---

## Comportements avancés

### RM-052e — Action : obligatoire et dépendance possible

Une action peut être marquée comme **obligatoire** (doit être exécutée) ou **optionnelle**.  
Une action peut aussi avoir une **dépendance** à une action précédente.

### RM-052f — Données custom stockables en JSON

En plus des données de chapitrage, le SDK permet de stocker des **données personnalisées au format JSON**.  
Chaque sauvegarde **écrase** les données précédentes.

### RM-052g — End() obligatoire pour démarrer une nouvelle session

La session doit être explicitement terminée via `End()`.  
Sans cet appel, l'utilisateur **reprend la même session**.

### RM-052h — Test en PlayMode Unity

Il est possible de tester le chapitrage en **PlayMode** directement dans Unity.  
Une session temporaire est créée et permet de vérifier la cohérence dans la console avant livraison.

---

## Sous-sections

- [Cycle de vie d'une session](session.md)
- [Stockage local & authentification](stockage.md)
- [Reporting](reporting.md)
- [Configuration & déploiement](configuration.md)

---


# Reporting dans le SDK `[v4.0]`

## RM-053h — Capture d'écran annotée

Le SDK expose une interface de reporting permettant d'annoter une capture d'écran :

- **Résolution** : 1280×720 px, PNG 24-bit
- **Commentaire texte**
- **Notation binaire** : `IsConform = true/false`
- **Dessin libre** sur l'image

**Flux d'envoi :**

1. `POST reporting/send` → retourne `NotationId`
2. `POST reporting/upload-screenshot` → upload du PNG

```
Source : inversive-sdk/Reporting/Runtime/Scripts/ScreenshotHandler.cs:41-100
         InversiveController.cs:12-46
```

---

## RM-053i — Validation de session obligatoire avant le reporting

Avant d'afficher l'interface de reporting, le SDK appelle `GET reporting/check` avec le `SessionId`.

!!! warning
    Si la session est invalide (expirée ou absente), l'interface de reporting est **désactivée** et aucune saisie n'est possible.

```
Source : inversive-sdk/Reporting/Runtime/Scripts/InversiveController.cs:48-73
```

---


# Cycle de vie d'une session SDK `[v4.0]`

## RM-053a — `Init()` obligatoire en premier

La méthode `Init()` doit être appelée **avant toute autre fonction du SDK**. Elle :

1. Récupère le `SessionId`
2. Charge le modèle d'expérience (cache ou serveur)
3. Initialise la session via l'endpoint `Initialize`

```
Source : inversive-sdk/Runtime/InversiveExperience.cs:256-316
```

---

## RM-053b — Récupération du `SessionId` selon la plateforme

| Plateforme | Source |
|------------|--------|
| **Unity Editor** | Requête `GET Account/get-session` avec AccessToken |
| **WebGL** | Dernier segment de l'URL |
| **Android** | Intent extras |
| **Autres** | Argument de ligne de commande `-session` |

```
Source : inversive-sdk/Runtime/InversiveExperience.cs:136-244
```

---

## RM-053c — Distinction `End()` / `Close()` / `Retry()`

| Méthode | Comportement |
|---------|-------------|
| **`End()`** | Termine **définitivement** la session, calcule `TimePlayed`, retourne le résumé de scores. **Doit être appelée avant `GetGlobalScore()`**. |
| **`Close()`** | Ferme sans terminer (session interrompue), met à jour `LastUpdateDate`. |
| **`Retry()`** | Réinitialise la session (efface les PlayerPrefs), récupère un nouveau `ExperienceSessionModel`. |

```
Source : inversive-sdk/Runtime/InversiveExperience.cs:466-568
```

---

## RM-053d — `ExecuteAction()` valide chapitre et action avant appel

Avant d'appeler l'endpoint `Action`, `ExecuteAction()` vérifie que le nom de chapitre et le nom d'action **existent dans `ExperienceModel`**.  
Si la validation échoue → l'appel est **abandonné**.  
La réponse contient `GlobalScore` et `ActionScore` via `ExperienceSessionActionSummaryModel`.

```
Source : inversive-sdk/Runtime/InversiveExperience.cs:358-422
```

---

## RM-053e — Gestion d'erreur HTTP 422

| Code | Message retourné |
|------|-----------------|
| **422** | `ParsingFailedMessage` (validation du modèle échouée) |
| Tout autre code | `CallFailedMessage` (générique) |

```
Source : inversive-sdk/Runtime/InversiveExperience.cs:398-401
```

---


# Stockage local & Authentification `[v4.0]`

## RM-053f — Cache local via PlayerPrefs

Le SDK utilise **PlayerPrefs** pour persister trois éléments localement :

| Clé | Contenu |
|-----|---------|
| `"ExperienceLocal"` | Modèle d'expérience complet (JSON) |
| `"ExperienceHead"` | Variante du modèle |
| `"ExperienceSession"` | Session utilisateur courante (JSON) |

Ce cache permet le **fonctionnement hors ligne** et la persistance entre redémarrages.

```
Source : inversive-sdk/Runtime/InversiveService.cs:46-96
```

---

## RM-053g — Authentification éditeur via `AccessToken.txt`

En mode **Unity Editor**, le SDK s'authentifie via un `AccessToken` stocké dans :

```
Assets/Inversive SDK/AccessToken.txt
```

Ce token est défini une fois via `InversiveService.SetAccessToken()`.

!!! warning
    Cette méthode est **inactive sur les builds de production**.

```
Source : inversive-sdk/Runtime/InversiveService.cs:7-44
```

---


# Personnalisation & Marque blanche

## RM-054 — 3 niveaux de personnalisation

La personnalisation visuelle peut être appliquée à :

| Niveau | Portée |
|--------|--------|
| **Toutes les organisations** | Ensemble de la company |
| **Une organisation spécifique** | Une seule `DeviceGroupSet` |
| **Un établissement spécifique** | Un seul `DeviceGroup` dans une organisation |

---

## RM-055 — L'aperçu en temps réel est interactif

!!! warning "Attention"
    L'aperçu de personnalisation est interactif.  
    Toute action effectuée dans cet aperçu est **réelle et prise en compte** sur la plateforme.

---

## RM-056 — Nom du produit et URL personnalisables

Chaque client peut définir :

- Un **nom de produit** personnalisé
- Une **URL d'accès dédiée** (marque blanche)

*Exemple : E-mersive BTP sur `https://e-mersive-btp.fr`*

---

## RM-056b — 13 options de personnalisation visuelle

| Option |
|--------|
| Couleur de fond |
| Couleur principale (boutons) |
| Logo |
| Favicon |
| Typographie |
| Arrondis |
| Couleur de fond de contenu |
| Couleur du texte |
| Fond du menu |
| Couleur du texte du menu |
| Couleur du menu actif |
| Bordure du menu |
| Couleur secondaire |

---

## RM-056c — Salons virtuels personnalisables par organisation/établissement

Les salons virtuels peuvent être personnalisés séparément pour chaque organisation ou établissement.  
Chaque salon peut avoir son propre **environnement 3D** (scène personnalisée ou scène par défaut).

---

## RM-056d — Raccourcis du salon virtuel : Pico uniquement

Les boutons de raccourcis dans le salon virtuel (paramétrage Wi-Fi, portail de connexion, zone de jeu) sont **uniquement disponibles sur les casques Pico**.

---


# Compatibilité des appareils

## Matrice de compatibilité

| Appareil | WebGL | VR filaire | APK AR | Casque sans fil |
|----------|-------|------------|--------|-----------------|
| **iPhone/iPad** | ✅ (pas de plein écran) | ❌ | ❌ | ❌ |
| **Android (smartphone)** | ✅ | ❌ | ✅ | ❌ |
| **PC Windows** | ✅ | ✅ (SteamVR) | ❌ | ❌ |
| **Casques autonomes** | ❌ | ❌ | ❌ | ✅ |

---

## RM-057 — Smartphones Apple : WebGL uniquement, pas de plein écran

Les smartphones Apple ne peuvent consommer que les expériences de type **Navigateur (WebGL)**.  
Le contenu WebGL **ne peut pas être affiché en plein écran** sur iOS.

---

## RM-058 — Smartphones Android

Les smartphones Android peuvent consommer :

- Les expériences **navigateur (WebGL)**
- Les expériences de **Réalité Augmentée** (téléchargement APK)

---

## RM-059 — PC Windows

Un PC Windows peut consommer :

- Les expériences **WebGL** via navigateur
- Les expériences **VR filaires** avec un casque SteamVR + **Immersive Catalog**

---

## RM-060 — Mise à jour casque nécessite internet

Si des informations ou paramètres d'un casque ont été modifiés, le casque doit être **connecté à internet** pour que les mises à jour soient appliquées.

---

## RM-060b — Casques VR sans fil supportés

| Constructeur | Modèles |
|---|---|
| **Meta** | Quest 2, Quest 3, Quest 3S |
| **Pico** | G3, 4 Enterprise, 4 Ultra Enterprise, Neo 3 Pro, Neo 3 Pro Eye |
| **HTC** | Vive Focus Vision, Vive XR Elite |

---

## RM-060c — Disparité fonctionnelle entre constructeurs

Les fonctionnalités disponibles peuvent **varier selon le constructeur**, en fonction de la capacité d'intégration que chaque constructeur met à disposition.

---

## RM-060d — Expériences WebGL : 3 formats de compression Unity supportés

| Format | Compression |
|--------|-------------|
| `br` | Brotli |
| `gz` | gzip |
| *(aucun)* | Sans compression |

---


# Événements publics `[v3.0]`

Les événements publics (`PublicEvent`) permettent d'ouvrir l'accès à des expériences **sans authentification préalable**, dans le cadre d'événements temporels (salons, démonstrations, conférences).

---

## RM-061 — Accès via header `PublicEventId`

Un accès à la plateforme peut être accordé en fournissant l'ID d'un événement public dans le header HTTP `PublicEventId`.  
Ce mécanisme permet à des **utilisateurs non authentifiés** de participer à un événement.

```
Source : inversive.web.api/Inversive.Web.Api/Auth/AuthorizeUserRightsAttribute.cs:143
```

---

## RM-062 — Événement accessible uniquement si non clôturé

Un accès via `PublicEventId` n'est accordé que si :

```
PublicEvent.ClosingDate >= DateTime.Now
```

Un événement passé sa date de clôture est **automatiquement inaccessible**.

---

## RM-063 — Sessions liées à un événement public

Lorsqu'une expérience est lancée dans le contexte d'un événement public, la session (`ExperienceSession`) est associée à cet événement via `PublicEventId`.  
Cela permet de **regrouper les statistiques par événement**.

---

## RM-064 — Politique d'archivage des sessions post-événement

Chaque événement public possède un flag `CanArchiveSession`.  
Lorsqu'il est activé, les sessions associées peuvent être **archivées après la clôture** de l'événement.

---


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

---


# Localisation & Langues `[v3.0]`

## RM-067 — 7 langues d'interface supportées

| Code | Langue |
|------|--------|
| `fr-FR` | Français |
| `en-US` | Anglais |
| `de-DE` | Allemand |
| `it-IT` | Italien |
| `es-ES` | Espagnol |
| `tr-TR` | Turc |
| `ru-RU` | Russe |

La langue par défaut d'une company est définie par son champ `LangId`.

```
Source : inversive.data/Inversive.Data/DatabaseContext.cs:481
```

---

## RM-068 — 100+ langues de contenu pour les expériences

Les expériences peuvent être décrites dans **plus de 100 langues de contenu** (`WorldLang`), chacune identifiée par un code culture (ex : `fr-FR`, `en-US`, `zh-CN`, `ar-SA`…).

Ces langues sont **distinctes des langues d'interface** et servent à localiser les titres, descriptions et contrôles des expériences.

```
Source : inversive.data/Inversive.Data/DatabaseContext.cs:501
```

---

## RM-069 — Traductions spécifiques par company

La table `EntityLang` permet à chaque company de **personnaliser les termes** utilisés dans l'interface (ex : renommer « Établissement » en « Site ») dans chacune des langues supportées.

Clé composite unique : `(EntityType, LangId, CompanyId)`

```
Source : inversive.data/Inversive.Data/Models/EntityLang.cs
```

---


# Temps réel & SignalR `[v3.0]`

La plateforme utilise **ASP.NET Core SignalR** avec **Redis** comme backplane pour la communication en temps réel entre le serveur, les casques, et les interfaces web de supervision.

---

## RM-070 — Connexions SignalR : authentification obligatoire

Toutes les connexions au hub SignalR (`DeviceStandaloneHub`) requièrent une **authentification**.  
Une connexion non authentifiée est immédiatement abandonnée via `Context.Abort()`.

```
Source : inversive.common.web/Inversive.Common.Web/Realtime/Hubs/DeviceStandaloneHub.cs
```

---

## RM-071 — Clés Redis scopées par niveau hiérarchique

Pour garantir le **multi-tenant**, les messages SignalR sont routés via des clés Redis scopées :

| Scope | Clé Redis |
|-------|-----------|
| Tous les casques VR | `vr:all` |
| Casques d'un établissement | `vr:dg:{deviceGroupId}` |
| Casques d'une organisation | `vr:dgs:{deviceGroupSetId}` |
| Casques d'une company | `vr:company:{companyId}` |
| Interface web d'une org | `web:dgs:{deviceGroupSetId}` |
| Interface web d'un étab. | `web:dg:{deviceGroupId}` |
| Interface web d'une company | `web:company:{companyId}` |
| Interface admin | `web:admin` |

```
Source : inversive.common.web/Inversive.Common.Web/Realtime/Hubs/Constants/RealtimeConstants.cs:43
```

---

## RM-072 — Méthodes temps réel supportées

| Méthode | Description |
|---------|-------------|
| `SendACK` | Acquittement d'une commande |
| `SendStatus` | Mise à jour du statut du casque |
| `ManageExperience` | Lancer/arrêter une expérience |
| `ManageCast` | Démarrer/arrêter le casting |
| `ManageKiosk` | Activer/désactiver le mode kiosque |
| `GetLogs` | Récupérer les logs du casque |
| `Signal` | Signal générique de communication |

```
Source : inversive.common.web/Inversive.Common.Web/Realtime/Hubs/Constants/RealtimeConstants.cs:22
```

---


# Headset Center `[v4.0]`

Application **WPF desktop** (.NET 10.0-windows) utilisée pour configurer les casques autonomes via USB/ADB et gérer la flotte en temps réel.

---

## Contraintes générales

### RM-073 — Instance unique obligatoire

Une seule instance du Headset Center peut s'exécuter simultanément.  
Un **mutex nommé** `VRCXP.Configurator` l'enforce. Toute seconde instance est bloquée.

```
Source : headset-center/HeadsetCenter/Program.cs:271
```

---

### RM-074 — Détection USB par VID/PID matériel

| Constructeur | VID |
|---|---|
| Meta/Oculus | `0x2833` |
| Pico | `0x2D40` |
| HTC | `0x0BB4` ou `0x18D1` |

Les PIDs spécifiques distinguent les modes (développeur, MTP, ADB).

```
Source : headset-center/HeadsetCenter/UsbDetection/Enums.cs:17
```

---

## Processus de configuration en 12 étapes

### RM-075

| Étape | Nom | Description |
|-------|-----|-------------|
| 1 | `ChooseTargetType` | Sélection du modèle de casque |
| 2 | `ChooseStructure` | Sélection company / organisation / établissement (filtrée par rôle) |
| 3 | `ChooseConfiguration` | Choix ou création d'un profil de casque |
| 4 | `ChooseParameters` | Configuration kiosk mode, mot de passe, Wi-Fi |
| 5 | `ChooseExperiences` | Sélection des expériences à déployer |
| 6 | `ChooseSummary` | Récapitulatif avant déploiement |
| 7 | `PlugHeadset` | Connexion physique USB avec ADB activé |
| 8 | `DownloadApps` | Téléchargement des binaires (service, lobby, expériences) |
| 9 | `DeployApps` | Installation via ADB |
| 10 | `ErrorReport` | Affichage des erreurs éventuelles |
| 11 | `NamingHeadset` | Saisie du nom du casque |
| 12 | `SuccessScreen` | Configuration terminée |

```
Source : headset-center/HeadsetCenter/Model/StepType.cs
```

---

### RM-076 — Mot de passe kiosk obligatoire si kiosk mode activé

Si le kiosk mode est activé lors de la configuration, `KioskModePassword` est **obligatoire**.  
Le bouton « Continuer » reste désactivé tant que le champ est vide.

---

### RM-077 — 2 chemins de déploiement

| Chemin | Endpoint | Détail |
|--------|----------|--------|
| **LobbyCore** | `POST /standalone` | Déploie BundleAssets, ThirdPartyApps, CustomLobby. Config via `config.json` poussé sur le casque. |
| **Standard** | `POST /standalone/create` | Pousse les payloads d'expériences et données de device via intent Android `com.vrcxp.service.entry.CONFIGURATION`. |

---

### RM-078 — Noms de packages système fixes

| Composant | Package Android |
|-----------|----------------|
| Service | `com.vrcxp.service` |
| Lobby | `com.vrcxp.home` |
| Video Player | `com.Inversive.VideoPlayer` |

---

### RM-079 — Seuls fr-FR et en-US poussés sur le casque

Lors de la configuration, seules les langues `fr-FR` et `en-US` sont transmises au casque (filtrées depuis l'API `/langs`).

---

### RM-080 — Retry de téléchargement avec validation MD5

- Validation MD5 après chaque téléchargement
- En cas d'échec : le téléchargement est relancé
- Paramètres configurables via `appsettings.json` :
  - `DownloadRetry.MaxRetries`
  - `RetryDelayMs`
  - `TimeoutMs`

---

## Gestion de la flotte

### RM-081 — Tri des appareils par priorité de statut

| Priorité | Statut |
|----------|--------|
| 1 (premier) | `InGame` — en cours d'expérience |
| 2 | `TurnedOn` — allumé |
| 3 | `Standby` — veille |
| 4 | `TurnedOff` — éteint |
| 5 (dernier) | `Disconnected` — déconnecté |

À égalité de statut, tri par **ordre alphabétique**.

---

### RM-082 — Conditions des actions groupées

| Action | Condition de disponibilité |
|--------|---------------------------|
| Lancer une expérience | Au moins 1 appareil `TurnedOn` |
| Arrêter une expérience | Au moins 1 appareil `InGame` ou avec un `CurrentExperienceName` |
| Kiosk Mode | **Tous** les appareils sélectionnés doivent être `TurnedOn` |

---

### RM-083 — Expériences de test triées en dernier

Dans l'étape de sélection des expériences à déployer, les expériences marquées `isTesting` sont systématiquement placées **en fin de liste**.

---


# Immersive Catalog `[v4.0]`

Application **WPF desktop** (.NET) permettant aux utilisateurs de naviguer dans le catalogue d'expériences, télécharger les binaires et lancer des expériences VR filaires ou standard sur PC.

---

## Contraintes générales

### RM-084 — Instance unique et récupération après crash

- Une seule instance autorisée (fermeture immédiate si doublon détecté)
- En cas de crash : redémarrage automatique avec l'argument `-fromcrash`, délai de **1 seconde** avant démarrage

---

### RM-085 — Mode hors ligne conditionnel

Le mode hors ligne est disponible **uniquement** si un token valide est présent en cache local.  
Sans token → erreur de connexion.  
En mode hors ligne, le dossier du catalogue doit aussi exister localement.

---

### RM-086 — Timeout HTTP de 72 heures pour les téléchargements

Le client HTTP pour les téléchargements a un timeout de **72 heures**, nécessaire pour les fichiers volumineux sur connexions lentes.

---

## Téléchargement & Installation

### RM-087 — 4 comportements selon le type d'expérience

| Type | Traitement |
|------|-----------|
| **VR** | Téléchargé en ZIP, extrait dans `Binaries/`, point d'entrée requis |
| **Video360** | Téléchargé comme `Binaries.mp4`, pas d'extraction |
| **ExternalApplication** | Téléchargé dans `Binaries/{EntryPoint}`, pas d'extraction |
| **Autres** (WebGL, Standard…) | Téléchargés en ZIP, extraits dans `Binaries/` |

Les binaires externes (sous-titres, audio) des Video360 sont téléchargés séparément depuis MinIO.

---

### RM-088 — Validation MD5 + retry max 3 tentatives

Après chaque téléchargement :

1. Hash MD5 calculé et comparé au hash fourni par l'API
2. En cas de divergence : retéléchargement avec **1 seconde de délai**
3. Maximum : **3 tentatives**

!!! info "Exemptions"
    Video360 et ExternalApplication sont **exemptés** de la validation MD5.

---

### RM-089 — Détection de mise à jour

| Champ | Déclenche |
|-------|-----------|
| `LastChangeDate` | Re-téléchargement des **assets** |
| `BuildLastUploadDate` | Re-téléchargement des **binaires** |

---

### RM-090 — Répertoires obsolètes supprimés automatiquement

Lors de la synchronisation avec l'API, les dossiers locaux d'expériences qui n'existent plus dans la réponse API sont **automatiquement supprimés**, ainsi que leurs entrées dans `Settings.ExperienceStatuses`.

---

### RM-091 — Écriture JSON atomique avec 10 tentatives max

Les fichiers `experience.json` sont écrits de manière **atomique** :

1. Écriture dans un fichier temporaire
2. Remplacement par `File.Replace()`
3. En cas d'échec (fichier verrouillé) : jusqu'à **10 tentatives** avant abandon

---

## Lancement & Intégration

### RM-092 — Protocole URL `vrcxp://`

L'Immersive Catalog est enregistré comme gestionnaire du protocole URL `vrcxp://`.

**Flux :**
1. URL `vrcxp://` écrit l'ID de l'expérience dans le registre Windows (`HKEY_CURRENT_USER\Software\Inversive\VRC`)
2. Un watcher de registre détecte le changement
3. Le catalogue se lance avec l'expérience cible

---

### RM-093 — Clic sur le X = masqué (pas fermé)

L'Immersive Catalog se comporte comme une application **systray**.  
Cliquer sur le bouton de fermeture **masque la fenêtre** sans quitter l'application.  
Une notification balloon s'affiche pendant **2 secondes**.

---


# Service Android — `inversive_service` `[v4.0]`

Service Android tournant en **arrière-plan** sur les casques autonomes.  
Gère la synchronisation des expériences, le reporting de télémétrie, l'authentification, le mode kiosk et les auto-mises à jour.

---

## Synchronisation des expériences

### RM-094 — 5 actions de synchronisation

Lors de chaque sync, chaque expérience est comparée avec le cache local et reçoit l'une des 5 actions :

| Action | Condition | Traitement |
|--------|-----------|------------|
| **Replace** | Version changée ET nom de package différent | Désinstaller l'ancien + installer le nouveau |
| **Upgrade** | Version changée (`lastUpload` ou `lastUpdate`) | Mettre à jour APK et/ou assets |
| **Install** | `isInstallRequired = true`, ou expérience de test non installée | Installer |
| **Uninstall** | `isUninstallRequired = true` | Désinstaller |
| **None** | Aucun changement détecté | Aucune action |

---

### RM-095 — Critères de mise à jour APK vs assets

| Critère | Déclenche |
|---------|-----------|
| `remote.lastUpload > local.lastUpload` | Mise à jour **APK** |
| `remote.lastUpdate > local.lastUpdate` | Mise à jour **assets** |

Ces deux mises à jour sont **indépendantes** et peuvent se produire simultanément.

---

### RM-096 — Sync hors ligne : cache local uniquement

Si le Wi-Fi est déconnecté lors d'une sync :

- Le service charge les expériences depuis la **base de données locale**
- **Aucune mise à jour ni installation** n'est effectuée

---

### RM-097 — Auto-mises à jour système uniquement si Wi-Fi connecté

Les auto-mises à jour du service, du lobby, du lecteur vidéo et du lecteur Uptale ne sont déclenchées que si le **Wi-Fi est connecté**.

Ordre de mise à jour (séquentiel) :
```
service → lobby → lecteur vidéo → lecteur Uptale
```

---

## Gestion des sessions

### RM-098 — Détection de fermeture d'expérience via UsageStats

| Paramètre | Valeur |
|-----------|--------|
| Polling interval | **1 seconde** sur une fenêtre de 10 secondes |
| Debounce | **500 ms** avant de reporter la fermeture |
| Seuil d'inactivité | L'expérience doit être en arrière-plan depuis **≥ 3 secondes** |
| Overlays système | Ignorés |

---

## Authentification & Réseau

### RM-099 — Authentification via OAuth 2.0 Device Code

**Flux :**
1. Le service reçoit un `userCode` + `deviceCode` avec expiration et intervalle de polling
2. Le `userCode` est diffusé à l'interface lobby pour affichage
3. Le service interroge le serveur à l'intervalle donné jusqu'à obtention du token

---

### RM-100 — Timeouts réseau

| Type de requête | Timeout |
|---|---|
| Requêtes API (request/connect/socket) | 30 secondes |
| Téléchargements de fichiers | 15 minutes |
| Polling DeviceCode | 60 secondes max |

---

### RM-101 — Reconnexion HeadsetCenter : backoff exponentiel

| Paramètre | Valeur |
|-----------|--------|
| Délai initial | 1 seconde |
| Facteur multiplicateur | ×2 |
| Maximum | 30 secondes |
| Nombre max de tentatives | **2** |

---

### RM-102 — Uptale : token SSO Bearer requis

Le lancement d'une expérience **Uptale** requiert un **token Bearer de type SSO** (pas un JWT standard).  
Si le Bearer disponible n'est pas de type SSO, le lancement échoue.  
Le token SSO est passé à l'URL de deep linking : `https://app.uptale.io/deeplinking/ExternalLaunch`

---

## Logs

### RM-103 — Logs uploadés en ZIP avec nettoyage automatique

1. Logs compressés dans un fichier **ZIP horodaté**
2. Uploadés via l'endpoint `/device/data`
3. Le fichier ZIP local est **supprimé après upload**

Répertoire des logs : `context.filesDir/logs`

---


# Lobby casque `[v4.0]`

Environnement **3D Unity** tournant sur les casques VR autonomes.  
Point d'entrée de l'utilisateur sur le casque : authentification, catalogue d'expériences, lancement, feedback, paramètres.

---

## Authentification sur casque

### RM-104 — OAuth 2.0 Device Code

Le lobby déclenche le flux OAuth 2.0 Device Code via `IPCBridge.Login()`.  
Un `userCode` est reçu et affiché à l'utilisateur pour qu'il le saisisse sur la plateforme web.  
L'authentification est confirmée par callback `OnLoginSuccess`.

---

### RM-105 — Connexion invité : email obligatoire

Pour se connecter en tant qu'invité :

- L'utilisateur doit saisir une **adresse email**
- L'email ne peut pas être vide ou composer uniquement d'espaces (`string.IsNullOrWhiteSpace`)
- Si la validation échoue → l'action est silencieusement abandonnée

---

## Catalogue d'expériences

### RM-106 — Seules les expériences installées ou à installer sont affichées

Le catalogue du lobby n'affiche que les expériences répondant à :

- `isInstalled == true` — APK installé sur le casque, **ou**
- `isInstallRequired == true` — installation requise pour accès

Les expériences non installées et non requises sont **ignorées**.

---

### RM-107 — Suppression des expériences disparues lors du refresh

Lors d'un refresh périodique, les expériences qui n'apparaissent plus dans la réponse du service sont **automatiquement retirées** de l'affichage.

---

## Lancement d'expériences

### RM-108 — Uptale : utilisateur authentifié obligatoire

Les expériences **Uptale** ne peuvent être lancées que si un utilisateur est authentifié (`CurrentUserProfile != null`).  
Les accès invité (par invitation) sont **insuffisants**.

---

### RM-109 — Onglet feedback visible seulement si `isTesting = true`

L'onglet de feedback (like/dislike + commentaire + captures d'écran) n'est visible que si `isTesting == true`.  
Pour les expériences validées (statut Actif), cet onglet **n'est pas affiché**.

---

## Paramètres & Comportement

### RM-110 — Composants UI filtrés par modèle de casque

Certains composants de l'interface peuvent être activés ou désactivés selon le `DeviceStandaloneType`.  
Ce mécanisme (`ActiveOnDeviceType`) permet d'afficher ou masquer des fonctionnalités spécifiques à un constructeur.

---

### RM-111 — Volume et luminosité : plage 0–100

Volume audio et luminosité configurables de **0 à 100**.

Conversion volume :
```
volumeAbsolu = (volume × volumeMax) / 100
```

---

### RM-112 — Lobby signale « prêt » après chargement des assets

Le lobby envoie `OnLobbyReady()` au service Android **uniquement après** que les assets (customisation, langue, logo) ont été chargés avec succès.  
Ce signal déclenche les opérations post-initialisation côté service.

---

### RM-113 — IPC fire-and-forget

La communication entre le lobby (Unity) et le service Android passe par un bridge JNI (`IPCFacade`).

- Chaque appel est **fire-and-forget** — le lobby ne bloque pas en attendant la réponse
- Les résultats arrivent via callbacks `OnMethodSuccess` / `OnMethodFailure`
- Les erreurs critiques déclenchent automatiquement un **popup d'erreur**

---


# Intégration LTI — `keycloak-lti-mapper` `[v4.0]`

Extension **Keycloak** (Java 17, Keycloak 26.0.0) qui enrichit les tokens OIDC avec les claims LTI 1.3 nécessaires à l'intégration Uptale.  
Déployée comme mapper Keycloak sur les clients LTI.

---

## Flux de lancement LTI

### RM-095a — Version LTI fixée à 1.3.0

Le mapper supporte exclusivement **LTI 1.3.0**.  
La version est systématiquement définie à `"1.3.0"` dans le token JWT généré.

---

### RM-095b — Détection automatique du type de message

| URL cible contient | `message_type` |
|---|---|
| `/lti/deeplinkSelect` | `LtiDeepLinkingRequest` |
| Autre | `LtiResourceLinkRequest` |

---

### RM-095c — Hiérarchie de résolution de l'URI cible

Ordre de priorité :

1. `lti_message_hint` décodé en Base64 *(source primaire)*
2. Paramètre URL `target_link_uri` *(flux standard)*
3. URI par défaut configuré (`default.target.uri`, défaut : `https://mydev.uptale.io/dashboard`)

---

### RM-095d — `lti_message_hint` encodé en Base64

Le `lti_message_hint` est un objet **JSON encodé en Base64**.  
Champs possibles : `target_link_uri`, `deep_link_return_url`, `data`, `device_groups`

Recherche dans l'ordre : AuthenticationSession notes → ClientSession notes → paramètres URL

---

## Mappage des rôles

| Groupe Keycloak | Rôle LTI attribué |
|---|---|
| `Uptale_Creator` | Instructor |
| Contient `admin` ou `owner` | Instructor **+** Administrator |
| Aucun des précédents | **Learner** (défaut) |

La comparaison des noms de groupe est **insensible à la casse**.

---

## Claims générées

### RM-095h — ResourceLink : UUID aléatoire

Pour `LtiResourceLinkRequest` : la claim `resource_link` contient un **UUID aléatoire** et le titre fixe `"Inversive Experience"`.  
Absente des `LtiDeepLinkingRequest`.

---

### RM-095i — Deep linking : `accept_multiple = true`

Pour `LtiDeepLinkingRequest` :

- `accept_multiple = true` obligatoire
- Types acceptés : `link`, `file`, `html`, `ltiResourceLink`, `image`
- Cibles : `iframe`, `window`, `embed`
- Le champ `data` contient le ticket ID

---

### RM-095j — Claim `context` : requiert `company_id` et `company_name`

La claim `context` n'est générée que si les attributs Keycloak `company_id` et `company_name` sont renseignés.  
Ces attributs doivent être synchronisés depuis le backend Inversive.

---

### RM-095k — Claim `inversive_device_groups`

Priorité de résolution : `lti_message_hint.device_groups` → attribut utilisateur Keycloak `device_groups` → liste vide

---

### RM-095l — Claims injectées dans 3 types de tokens

Les claims LTI sont présentes dans :

- **ID Token**
- **Access Token**
- **UserInfo Token**

---

## Configuration

### RM-095m — Deployment ID

Configurable via `deployment.id`.  
Valeur par défaut : `inversive-uptale-1`

### RM-095n — URL de callback deep linking

Résolution : `lti_message_hint` → propriété mapper `platform.callback.url`  
Si aucune source ne la fournit → avertissement critique journalisé, mais le processus continue.

---


# API Standalone — `inversive.web.standalone` `[v4.0]`

API dédiée aux casques VR autonomes.  
Miroir fonctionnel de `inversive.web.api` avec une couche d'authentification spécifique aux appareils (`DeviceStandalone`) et des endpoints orientés hardware.

---

## Authentification des appareils

### RM-114 — Authentification via header `DeviceId`

Les casques autonomes s'authentifient en envoyant leur identifiant dans le header HTTP `DeviceId`.

---

### RM-115 — Appareils archivés rejetés

Un appareil `IsArchived = true` est rejeté avec le message :  
`"Unknown or deactivated device"`

---

### RM-116 — Module `Standalone` requis

Un casque ne peut accéder à l'API que si sa company a le module `Standalone` activé.  
La vérification traverse la hiérarchie : `DeviceStandalone → DeviceGroup → Company → Modules`

---

### RM-117 — JWT casque : 2 claims

| Claim | Valeur |
|-------|--------|
| `NameIdentifier` | `deviceId` |
| `UserType` | `"DeviceStandalone"` |

---

### RM-118 — Refresh tokens expirés purgés à chaque auth

À chaque authentification, les refresh tokens expirés ou révoqués sont **automatiquement supprimés** (purge en ligne).

---

### RM-119 — Refresh token : 64 octets crypto-aléatoires

Généré via `RandomNumberGenerator.GetBytes(64)` converti en Base64.

---

### RM-120 — Rotation du refresh token

Lors du renouvellement : l'ancien token est marqué `Revoked = DateTime.UtcNow` **avant** l'émission du nouveau.  
Un token révoqué ne peut pas être réutilisé.

---

### RM-121 — Nettoyage des refresh tokens révoqués

Un service de fond (`RefreshTokenCleanupService`) supprime les tokens expirés/révoqués :

- Une fois **au démarrage de l'application**
- Puis toutes les **24 heures**

---

### RM-122 — Sources d'authentification Keycloak autorisées

| Source | Condition |
|--------|-----------|
| `local` | Toujours autorisée |
| `workspace` | Toujours autorisée |
| `google` | Uniquement si domaine `vr-connection.com` |

Toute autre source Google est **rejetée**.

---

### RM-123 — Auto-provisioning des utilisateurs Google `vr-connection.com`

Un utilisateur s'authentifiant via Google avec le domaine `vr-connection.com` est **automatiquement créé** en base s'il n'existe pas.

---

### RM-124 — UserType par défaut selon l'environnement

| Environnement | UserTypeId | CompanyId |
|---|---|---|
| Production | `2` (CompanyOwner) | `1` |
| Dev/Staging | `1` (Admin) | `null` |

---

### RM-125 — Token SignalR via query string

Pour les connexions WebSocket au hub `/devicehub`, le token est extrait du paramètre URL `access_token`  
(et non du header `Authorization`, conformément aux contraintes WebSocket des navigateurs).

---

## Gestion des expériences sur casque

### RM-126 — Normalisation des emails d'invitation

Avant vérification, l'email est normalisé :

1. `Trim()`
2. Suppression des caractères de **largeur zéro** (`U+200B`)
3. Conversion en **minuscules**

!!! tip
    Évite les rejets silencieux dus à des caractères invisibles copiés-collés.

---

### RM-127 — Suivi de l'installation

| Endpoint | Action |
|----------|--------|
| `PATCH /{experienceId}/installed` | Marque l'expérience comme installée |
| `PATCH /{experienceId}/removed` | Marque l'expérience comme désinstallée |

---

### RM-128 — Sessions liées à une invitation

Au démarrage d'une session, le casque transmet le `InvitationId` dans le header HTTP.  
Cet identifiant est stocké dans le modèle de session pour traçabilité.

---

### RM-129 — 2 modes de vérification d'invitation

| Mode | Méthode HTTP | Contexte |
|------|-------------|----------|
| `CheckInvitationByEmail` | POST avec email | Appareil = `Client.Device` |
| `CheckInvitationByCredentials` | GET sans paramètre | Appareil = `Client.Human` |

---

## URLs présignées (MinIO)

| Ressource | Expiration |
|-----------|-----------|
| Expériences / VideoPlayer / services Android | **1 000 secondes** (~16 min) |
| APK via SetupController | **60 secondes** |

---

### RM-132 — `versionCode` incrémenté de +1

Lors du retour des informations de version du service Android, le `versionCode` est systématiquement incrémenté de `+1` pour **forcer la mise à jour** côté Android.

---

## Fonctionnalités additionnelles

### RM-133 — Credentials TURN pour WebRTC

Le `TurnController` génère des credentials TURN temporaires sécurisés pour établir des connexions WebRTC peer-to-peer (Cast, visioconférence).

---

### RM-134 — Version du lobby selon le type d'appareil

La version de lobby correspond au `DeviceStandaloneType` de l'appareil.  
Si aucune version spécifique n'existe → fallback `DefaultLobby`.

---

### RM-135 — Flag `IsVanilla`

| Valeur | Signification |
|--------|---------------|
| `true` | Version générique plateforme |
| `false` | Personnalisé par la company |

---

### RM-136 — Endpoint de logs : SuperAdmin uniquement

`GET /device/data` nécessite l'autorisation `SuperAdmin`.  
Retourne une URL présignée MinIO pour télécharger les logs zippés de l'appareil.

---


# VideoPlayer Android `[v4.0]`

Application **Unity Android** déployée sur les casques VR autonomes pour la lecture de vidéos 360°.  
Aucune communication API avec la plateforme — **fonctionnement entièrement local**.

---

## RM-137 — Paramètres de lancement via Intent Android

Le VideoPlayer reçoit sa configuration au démarrage via l'Intent Android, clé `videoPlayerData` (JSON) :

| Champ | Type | Description |
|-------|------|-------------|
| `experienceId` | `int` | Identifiant de l'expérience |
| `angle` | `string` | Format de projection vidéo |

Ces paramètres sont fournis par le **service Android** lors du lancement.

---

## RM-138 — Chemin de stockage des vidéos

```
/storage/emulated/0/Download/video360/{experienceId}/{experienceId}.mp4
```

Ce chemin est construit par `StaticMethods.GetGamePath(experienceId)` et doit être respecté par tout composant qui dépose les vidéos.

---

## RM-139 — Fichiers audio multilingues

Pattern : `{experienceId}_LANGUE.{extension}`

| Extension | Type |
|-----------|------|
| `.amb` | Audio ambisonic spatial |
| `.mp3` | MP3 |
| `.wav` | WAV |
| `.flac` | FLAC |
| `.tbe` | TBE |

Un audio par défaut intégré à la vidéo est toujours disponible.

---

## RM-140 — Fichiers de sous-titres

Pattern : `{experienceId}_LANGUE.srt`

**Seul le format SRT est supporté.**  
L'absence de fichier `.srt` signifie qu'aucun sous-titre n'est disponible.

---

## RM-141 — 6 formats de projection vidéo 360°

| Format | ID interne | Rendu |
|--------|-----------|-------|
| 180° | `2` | Mono |
| 360° | `3` | Mono |
| 360° Haut-Bas | `3` | Stéréo |
| 360° Gauche-Droite | `5` | Stéréo |
| 180° Haut-Bas | `11` | Stéréo |
| 180° Gauche-Droite | `13` | Stéréo |

Les formats **stéréo** activent le rendu binoculaire (deux yeux).  
Les formats **mono** n'utilisent qu'un seul rendu.

---

## RM-142 — Audio `.amb` active le mode spatial automatiquement

| Extension | `spatialize` Unity |
|-----------|-------------------|
| `.amb` | `true` (son spatial 3D) |
| Autres formats | `false` |

---


# Application Web — Blazor `[v4.0]`

Frontend **Blazor WebAssembly**.  
Couvre : gestion des sessions, validations de formulaire, routage, stockage local et temps réel.

---

## Gestion des tokens & sessions

### RM-143 — Délai de grâce d'expiration : 30 secondes

Un token d'accès est considéré expiré **30 secondes avant** son expiration réelle.  
S'applique au cache mémoire **et** au localStorage.

---

### RM-144 — Renouvellement du token : mutex (pas de concurrence)

Flag `_externalTokenRefreshInProgress` : si un renouvellement est en cours, les autres demandes reçoivent immédiatement `TokenResult.Unavailable()`.

---

### RM-145 — Ordre de résolution du token

1. Cache **mémoire**
2. **localStorage** (token externe)
3. Token **OIDC** standard

!!! warning
    Si un token OIDC valide **et** un token externe sont tous deux présents → le localStorage est nettoyé et l'utilisateur est redirigé vers la connexion.

---

### RM-146 — Réhydratation de session multi-onglet

Si un nouvel onglet détecte un token valide mais un localStorage vide (`currentVersion` ET `userRight` absents) → `GetUserProfileInfos` est rappelé pour reconstruire le contexte.

---

## Authentification & Autorisation

### RM-147 — Sources d'authentification autorisées

| Source | Condition |
|--------|-----------|
| `local` | Toujours autorisée |
| `google` | Domaine `vr-connection.com` uniquement |
| `EducNat` | Toujours autorisée |

Toute autre source → redirection vers `unauthorized`.

---

### RM-148 — Mode `queryToken` : bypass OIDC

Un paramètre GET `queryToken` valide permet une authentification sans flux OIDC.  
Tout token OIDC existant est nettoyé. Utilisé pour les **intégrations externes**.

---

### RM-149 — Vérification hiérarchique des rôles

```
CurrentUserType.BaseUserType <= RequiredUserType
```

Un Admin (1) accède à tout ce qui requiert CompanyOwner (2), DGSManager (4), etc.

---

### RM-150 — Admin exempté de la vérification de module

Les utilisateurs `Admin` contournent la vérification d'activation de module.  
Un Admin voit **tous les modules** indépendamment de leur état.

---

### RM-151 — Opérateurs AND/OR pour les vérifications multi-droits

| Opérateur | Condition |
|-----------|-----------|
| `Or` (défaut) | Au moins un droit requis |
| `And` | Tous les droits requis |

---

### RM-152 — Impersonation : Producer et User exclus

La fonctionnalité d'impersonation est réservée aux rôles :  
Admin · CompanyOwner · DeviceGroupSetManager · DeviceGroupManager

---

## Validations de formulaires

### RM-153 — Asset : company obligatoire pour les non-Admin

Lors de la création d'un asset, si le créateur n'est pas Admin, une **company doit être sélectionnée**.

### RM-154 — Tags d'asset obligatoires si catégorie sélectionnée

Si une `AssetCategory` est sélectionnée ET que des tags sont disponibles → **au moins un tag doit être coché**.

### RM-155 — Nombre de joueurs : plage [1, 100]

`MinPlayerCount` et `MaxPlayerCount` sont contraints à **[1, 100]**.

### RM-156 — Formats de binaires par type d'expérience

| Type | Extensions acceptées |
|------|---------------------|
| Vidéo 360 | `.mov`, `.mp4`, `.avi`, `.wmv`, `.flv`, `.3gp`, `.webm` |
| VR / WebGL | `.zip` uniquement |
| Audio externe | `.mp3` uniquement |
| Vidéo externe | `.mp4` uniquement |

### RM-157 — `ExternalApplicationType` obligatoire

Si type = `ExternalApplication` → `ExternalApplicationType` doit être sélectionné.

### RM-158 — Conversion automatique SRT → VTT

Lors de l'upload d'un fichier `.srt` : le navigateur le convertit automatiquement en `.vtt` et uploade les **deux fichiers**.  
L'opération est transparente pour l'utilisateur.

### RM-159 — Limite d'upload client : 2 Go

Tout fichier dépassant **2 Go** est rejeté avant l'ouverture du stream.

### RM-160 — Screenshots d'expérience : formats image uniquement

Extensions acceptées : `.jpg`, `.png`, `.jpeg`, `.svg`  
Tout autre format → rejeté avec `"Only images are allowed"`

### RM-161 — Nombre maximum de screenshots d'asset

| Type d'asset | Max screenshots |
|---|---|
| Asset2D, VFX, Video, Audio | **1** |
| Package, Asset3D | **5** |

---

## Routage & Navigation

### RM-162 — Page d'accueil selon le rôle

| Rôle | Page d'accueil |
|------|----------------|
| Producer | `/experience/manage` |
| User | `/experience/catalog-webgl` (si module actif) ou `/experience/index` |
| Admin | `/admin/index` |
| CompanyOwner/Managers | Analysé par droits (DeviceGroup, Experience, User, Device) |

### RM-163 — Appareils filtrés par compatibilité binaire

Lors de la sélection d'appareils pour lancer une expérience, seuls les appareils dont le `DeviceStandaloneType` correspond aux types de binaires **Standalone** ou **Video360** de cette expérience sont affichés.

### RM-164 — URL de retour sauvegardée avant redirection

L'URL courante est sauvegardée en localStorage avant redirection vers la connexion.  
Après authentification → l'utilisateur est renvoyé vers cette URL.

### RM-165 — Module LemonLearning

Si la company a le module LemonLearning activé → données utilisateur (nom, email, rôle, OFA, CFA) transmises à un **script tiers chargé de manière asynchrone**.

---

## Temps réel — SignalR côté client

### RM-166 — Reconnexion : backoff exponentiel 2ⁿ secondes, max 60s

- Délai : **2ⁿ secondes** (n = nombre de tentatives)
- Maximum : **60 secondes**
- Politique : **infinie** (`InfiniteRetryPolicy`)

### RM-167 — Suivi des ACK par commande

Chaque commande envoyée est enregistrée dans un `AckTracker`.  
Lorsqu'un ACK est reçu → il est associé à la commande originale.

### RM-168 — État de casting par appareil

Un objet `DeviceCastingState` maintient le flag `IsCasting` par appareil.  
À l'arrêt du cast → événement `StopCast` transmis via SignalR.

---

## Stockage local (localStorage)

### RM-169 — Profil utilisateur mis en cache

Données stockées après authentification :

`userType` · `companyId` · `deviceGroupId` · `deviceGroupSetId` · `email` · `userName` · `profilePicture` · `theme` · modules activés (`StandaloneModule`, `AssetsLibraryModule`, `CatalogWebGLModule`, `DoubleLogoModule`, `StatisticsModule`) · `isAdmin` · `IsImpersonated`

### RM-170 — Admin : `StatisticsModule = true` toujours

Pour les utilisateurs Admin, `StatisticsModule = true` est défini **inconditionnellement**.

### RM-171 — Likes mis en cache

Clé localStorage : `ExperienceUserLikes`  
À chaque modification → le cache est mis à jour (remplacement si existant, ajout sinon).

### RM-172 — Langue : défaut `fr-FR`, persistée

La langue est lue depuis localStorage.  
Si absente → `fr-FR` par défaut.  
Sauvegardée via `blazorCulture.set`.

### RM-173 — Panier d'assets persisté

La sélection d'assets dans le panier est sauvegardée et restaurée à la session suivante.

### RM-174 — Acceptation des CGU persistée

L'état d'acceptation des CGU et sa date sont stockés en localStorage (évite de reproposer à chaque reconnexion).

---

## Recherche

### RM-175 — Suppression des diacritiques

Les requêtes de recherche sont normalisées côté client : les accents sont supprimés (`é → e`, `à → a`, etc.) avant comparaison.

---

## Comportements UI

### RM-176 — Message de succès masqué après 2 secondes

Après une action réussie → message affiché puis masqué automatiquement après **2 secondes** via `Task.Delay(2000)`.

### RM-177 — Uptale : vérification à 3 niveaux

Le module Uptale est actif uniquement si **les 3 conditions sont réunies** :

1. La company a le module Uptale activé
2. Le DeviceGroupSet a `IsUptaleEnabled = true`
3. Le DeviceGroup hérite de `IsUptaleEnabled` via son DGS

Un seul niveau manquant → **module désactivé** pour l'utilisateur.

---


# Notes de conformité

Cette annexe documente les **écarts identifiés entre le document v2.0 et le code source**.  
Ces écarts ne représentent pas des bugs mais des divergences sémantiques ou des détails d'implémentation à connaître.

---

## CONF-001 — Politique de mot de passe (→ RM-002)

| | Détail |
|-|--------|
| **Document v2.0** | Caractères spéciaux autorisés : `` $@&()[]{}=#,-!?+/£€% `` |
| **Code** | Aucune validation de pattern au niveau des modèles API. Le champ `Password` est marqué `[Required]` uniquement. |
| **Explication** | La politique de mot de passe est appliquée par **Keycloak** (Identity Provider), pas par l'API Inversive. |
| **Action recommandée** | Vérifier la configuration Keycloak pour s'assurer qu'elle correspond aux règles documentées. |

---

## CONF-002 — Validité d'une invitation de test (→ RM-017)

| | Détail |
|-|--------|
| **Document v2.0** | « L'invitation se fait par email avec un lien d'accès valable 15 jours par défaut. » |
| **Code** | `invitation.Expiration < DateTime.Today` — le **jour d'expiration est inclus** jusqu'en fin de journée (23:59:59). |
| **Explication** | La comparaison utilise `DateTime.Today` (date sans heure). Si l'expiration est fixée à J+15, l'invitation reste valide jusqu'à la fin de la journée J+15. |
| **Action recommandée** | Mettre à jour RM-017 : « valable 15 jours par défaut, expiration incluse jusqu'en fin de journée. » |

---

## CONF-003 — Mode Kiosque et Pico (→ RM-036)

| | Détail |
|-|--------|
| **Document v2.0** | « Les actions Mode Kiosk, Éteindre et Redémarrer sont disponibles uniquement pour les casques Pico Enterprise. » |
| **Code** | Le champ `KioskMode` est présent sur l'entité `DeviceStandalone` pour **tous les modèles de casques**, sans filtrage par constructeur dans le modèle de données. |
| **Explication** | La restriction Pico est probablement appliquée au niveau de l'**interface utilisateur** (UI) ou au niveau des commandes envoyées au casque. |
| **Action recommandée** | Confirmer si la restriction est côté API (à implémenter) ou purement côté UI (à documenter ainsi). |

---

## CONF-004 — Terminologie métier vs technique

| Terme métier (document v2.0) | Terme technique (code) |
|------------------------------|------------------------|
| Gestionnaire d'organisation | `DeviceGroupSetManager` |
| Gestionnaire d'établissement | `DeviceGroupManager` |
| Organisation | `DeviceGroupSet` |
| Établissement | `DeviceGroup` |
| Apprenant | `User` (BaseUserType) |
| Producteur de contenu | `Producer` (BaseUserType) |
| Administrateur | `CompanyOwner` (BaseUserType) |
| Super Admin | `Admin` (BaseUserType) |

Ces deux vocabulaires coexistent dans la plateforme.  
Le document v2.0 utilise le vocabulaire **métier** ; le code utilise la terminologie **technique**.  
Les deux sont valides dans leurs contextes respectifs.

---


# Annexes

- [Notes de conformité](conformite.md) — Écarts entre le document v2.0 et le code source
- [Table de correspondance terminologique](terminologie.md) — Termes métier ↔ termes techniques

---


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

---

