# Formats & Binaires

## RM-026 — Un binaire par type d'appareil

Pour une même expérience disponible sur plusieurs types d'appareils, un **binaire distinct** doit être fourni pour chaque type.

| Type d'appareil | Format binaire |
|-----------------|----------------|
| Casque sans fil | `.apk` |
| Navigateur | WebGL (`.zip`) |
| PC filaire (SteamVR) | `.exe` |

---

## RM-027 — Persistance de session cross-appareils via SDK

Une expérience intégrant le SDK Inversive peut :

- **Enregistrer la progression** d'un utilisateur
- Lui permettre de **reprendre sur un autre appareil** exactement là où il s'était arrêté

La session reste ouverte jusqu'à l'appel de la fonction `End()`.

---

## RM-028 — Point d'entrée obligatoire pour les exécutables multi-.exe

Si un binaire contient **plusieurs `.exe`**, l'auteur doit obligatoirement en sélectionner un comme **point d'entrée** lors de la finalisation de l'expérience.

---

## RM-028b — Modèle de casque obligatoire pour les expériences sans fil

Lors de l'ajout d'un binaire de type « Casque sans fil » :

- Le **modèle de casque cible** doit être sélectionné
- Un même contenu peut être décliné pour **plusieurs modèles** de casques

---

## RM-029 — Suppression = envoi en archives

Une expérience supprimée n'est **pas définitivement effacée** : elle est déplacée dans la section **Archives** d'où elle peut être désarchivée.

---

## RM-029b — Modification d'une expérience

La modification d'une expérience ouvre un formulaire **identique à la création**.  
Toute modification doit être validée via **« Enregistrer les modifications »**.
