# Rôles personnalisés & Usurpation

## RM-009 — Impossible d'accorder des droits supérieurs aux siens

Un utilisateur ne peut pas créer de rôle personnalisé avec des droits supérieurs aux siens.  
Un gestionnaire ne peut pas accorder des droits d'administrateur.

---

## RM-010 — Usurpation de rôle limitée vers le bas

La fonctionnalité d'**usurpation de rôle** (disponible pour Administrateurs et Gestionnaires) ne permet pas d'usurper un rôle supérieur au sien dans la matrice des droits.

---

## RM-011 — Actions d'usurpation sont effectives

!!! warning "Attention"
    Toutes les actions effectuées durant une session d'usurpation de rôle sont **réelles et prises en compte** sur la plateforme (modification de disponibilité, soumission d'expérience, etc.).
