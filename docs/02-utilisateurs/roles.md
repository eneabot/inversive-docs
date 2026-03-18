# Hiérarchie des rôles

## RM-008 — Héritage des droits vers le haut

Les droits accordés à un rôle doivent **obligatoirement être accordés aux rôles de niveau supérieur**.  
Ce qu'un gestionnaire peut faire doit pouvoir être fait par un administrateur.

---

## RM-008b — Périmètre du Gestionnaire d'organisation

Le gestionnaire d'organisation peut :

- ✅ Gérer les expériences qu'**il a créées**, au sein de son organisation
- ✅ Modifier la disponibilité des expériences dans les établissements de son organisation
- ❌ Créer des utilisateurs
- ❌ Gérer des expériences créées par d'autres

---

## RM-008c — Périmètre du Gestionnaire d'établissement

Le gestionnaire d'établissement peut :

- ✅ Gérer les expériences qu'**il a créées** au sein de son établissement
- ✅ Gérer les utilisateurs dans son unique établissement
- ❌ Intervenir dans d'autres établissements
