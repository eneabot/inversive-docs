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
