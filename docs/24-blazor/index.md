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
