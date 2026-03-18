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
