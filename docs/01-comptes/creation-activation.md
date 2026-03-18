# Création & Activation de compte

## RM-001 — Activation par email obligatoire

Tout nouveau compte doit être activé via un lien envoyé par email.  
L'utilisateur **ne peut pas se connecter** avant d'avoir cliqué sur ce lien.

---

## RM-002 — Politique de mot de passe

Le mot de passe doit comporter **au minimum 8 caractères** incluant :

- au moins une **majuscule**
- au moins une **minuscule**
- au moins un **chiffre**
- au moins un **caractère spécial** parmi : `` $@&()[]{}=#,-!?+/£€% ``

!!! warning "Note de conformité — CONF-001"
    Cette validation est appliquée par **Keycloak** (Identity Provider), pas au niveau de l'API.  
    L'API se contente de marquer le champ `Password` comme `[Required]`.  
    → Vérifier la configuration Keycloak pour s'assurer qu'elle correspond à ces règles.

---

## RM-003 — Activation différée possible

La date d'envoi du mail d'activation peut être **programmée à une date future**.  
L'utilisateur ne peut pas activer son compte avant réception de ce mail.

---

## RM-004 — Import en masse limité aux apprenants

Seuls les utilisateurs de type **Apprenant** peuvent être importés en masse via fichier Excel (`.xlsx`).  
Les autres rôles doivent être créés individuellement.

---

## RM-004b — Rapport d'erreur à l'import en masse

Lors d'un import en masse :

- Si des utilisateurs n'ont pas pu être créés → un **encart orange** liste les raisons d'échec
- Les utilisateurs valides sont **tout de même importés** (pas de rollback total)

---

## RM-004c — Utilisateurs importés avec date future : statut non activé

Si une date d'activation future est choisie lors de l'import en masse :

- Les utilisateurs sont ajoutés à la liste
- Ils restent **non activés** jusqu'à la date programmée ou une action manuelle
