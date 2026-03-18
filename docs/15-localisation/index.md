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
