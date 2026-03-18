# Disponibilité des expériences

## RM-022 — 3 niveaux de déploiement obligatoires

Pour qu'une expérience soit accessible aux utilisateurs finaux, **3 étapes successives** sont requises :

```
1. Activation par l'administrateur au niveau Organisation/Autre structure
        ↓
2. Mise à disposition au niveau Établissement par le gestionnaire
        ↓
3. Accès effectif aux utilisateurs et casques de l'établissement
```

---

## RM-023 — Désactivation au niveau Organisation : cascade sur les établissements

Si un contenu est retiré d'une Organisation, il est **automatiquement et immédiatement** retiré de **tous les Établissements** qui composent cette Organisation.

---

## RM-024 — Expérience désactivée = visible uniquement par les admins

Une expérience non activée dans une Organisation est uniquement visible par les **administrateurs**.  
Les gestionnaires et apprenants n'y ont pas accès.

---

## RM-025 — Expériences sur casque filtrées par compatibilité + établissement

Dans un profil de casque, les expériences disponibles à la sélection sont celles qui sont :

- ✅ **Compatible** avec le modèle de casque
- ✅ **Disponible** dans l'établissement associé au profil

Les deux conditions doivent être réunies simultanément.

---

## RM-025b — Disponibilité au niveau Organisation ne suffit pas

!!! warning
    Activer une expérience dans une Organisation **ne la rend pas directement accessible** aux utilisateurs finaux.  
    Elle doit encore être mise à disposition au niveau de chaque **Établissement** concerné.
