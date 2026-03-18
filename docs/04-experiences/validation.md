# Validation & Tests

## RM-016 — Deux voies de finalisation

Lors de la création, l'auteur choisit entre :

| Voie | Statut résultant |
|------|-----------------|
| **Valider sans tester** | → **Actif** directement |
| **Inviter des testeurs** | → **À valider** (phase de test) |

---

## RM-017 — Invitation de testeurs sans compte plateforme

Il n'est **pas nécessaire** d'avoir un compte sur la plateforme pour tester une expérience en tant qu'invité.  
L'invitation se fait par email avec un lien d'accès valable **15 jours par défaut**.

!!! note "Note de conformité — CONF-002"
    Le code vérifie `Expiration < DateTime.Today`, ce qui inclut le **jour d'expiration jusqu'en fin de journée** (23:59:59).

---

## RM-018 — Durée de validité d'une invitation éditable

La période de validité (15 jours par défaut) peut être éditée à tout moment par l'administrateur.

---

## RM-019 — Retour de test possible après validation

Même après validation (statut **Actif**), il reste possible d'inviter des utilisateurs à tester l'expérience.

---

## RM-020 — Cycle refus / correction / re-validation

1. L'expérience est **refusée** → le producteur reçoit un email de notification
2. Le producteur **corrige** les binaires → statut repasse à **À valider**
3. Nouveau cycle de validation

Les informations de refus sont visibles dans l'onglet **« Suivi des refus »**.

---

## RM-021 — Retour de test : commentaire + jusqu'à 5 images

Un testeur peut approuver ou refuser une expérience en joignant :

- Un **commentaire**
- Jusqu'à **5 images**

Un administrateur refusant une expérience peut de même justifier sa décision (commentaire + 5 images).

---

## RM-021b — Test sur casque autonome : casque doit appartenir à la flotte

Pour ajouter une expérience en test sur un casque autonome, ce casque doit **impérativement appartenir à la flotte** du compte administrateur.  
Sinon, il n'apparaît pas dans la liste de sélection.

---

## RM-021c — Expérience en test sur casque : affichage en noir et blanc

Sur un casque, une expérience en cours de test s'affiche **en noir et blanc**.  
Pour la lancer, l'utilisateur doit :

- Être connecté avec un compte ayant accès aux expériences de test, **ou**
- Avoir été invité

---

## RM-021d — Commentaires depuis l'expérience elle-même

Il est possible d'effectuer des commentaires de test **depuis l'intérieur de l'expérience**, à condition que :

- L'expérience soit développée pour **Pico Enterprise**
- Elle intègre le **SDK Inversive**

---

## RM-021e — Retours classés par ordre alphabétique puis chronologique

Dans l'interface de consultation des retours de test, le tri est :

1. Ordre alphabétique du testeur
2. Ordre chronologique du plus récent au plus ancien
3. Par type de média
