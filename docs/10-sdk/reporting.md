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
