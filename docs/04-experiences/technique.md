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
