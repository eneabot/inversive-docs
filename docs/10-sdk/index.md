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
