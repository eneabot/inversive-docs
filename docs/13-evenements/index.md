# Événements publics `[v3.0]`

Les événements publics (`PublicEvent`) permettent d'ouvrir l'accès à des expériences **sans authentification préalable**, dans le cadre d'événements temporels (salons, démonstrations, conférences).

---

## RM-061 — Accès via header `PublicEventId`

Un accès à la plateforme peut être accordé en fournissant l'ID d'un événement public dans le header HTTP `PublicEventId`.  
Ce mécanisme permet à des **utilisateurs non authentifiés** de participer à un événement.

```
Source : inversive.web.api/Inversive.Web.Api/Auth/AuthorizeUserRightsAttribute.cs:143
```

---

## RM-062 — Événement accessible uniquement si non clôturé

Un accès via `PublicEventId` n'est accordé que si :

```
PublicEvent.ClosingDate >= DateTime.Now
```

Un événement passé sa date de clôture est **automatiquement inaccessible**.

---

## RM-063 — Sessions liées à un événement public

Lorsqu'une expérience est lancée dans le contexte d'un événement public, la session (`ExperienceSession`) est associée à cet événement via `PublicEventId`.  
Cela permet de **regrouper les statistiques par événement**.

---

## RM-064 — Politique d'archivage des sessions post-événement

Chaque événement public possède un flag `CanArchiveSession`.  
Lorsqu'il est activé, les sessions associées peuvent être **archivées après la clôture** de l'événement.
