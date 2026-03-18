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
