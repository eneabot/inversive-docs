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
