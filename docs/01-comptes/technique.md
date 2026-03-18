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
