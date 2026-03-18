# Temps réel & SignalR `[v3.0]`

La plateforme utilise **ASP.NET Core SignalR** avec **Redis** comme backplane pour la communication en temps réel entre le serveur, les casques, et les interfaces web de supervision.

---

## RM-070 — Connexions SignalR : authentification obligatoire

Toutes les connexions au hub SignalR (`DeviceStandaloneHub`) requièrent une **authentification**.  
Une connexion non authentifiée est immédiatement abandonnée via `Context.Abort()`.

```
Source : inversive.common.web/Inversive.Common.Web/Realtime/Hubs/DeviceStandaloneHub.cs
```

---

## RM-071 — Clés Redis scopées par niveau hiérarchique

Pour garantir le **multi-tenant**, les messages SignalR sont routés via des clés Redis scopées :

| Scope | Clé Redis |
|-------|-----------|
| Tous les casques VR | `vr:all` |
| Casques d'un établissement | `vr:dg:{deviceGroupId}` |
| Casques d'une organisation | `vr:dgs:{deviceGroupSetId}` |
| Casques d'une company | `vr:company:{companyId}` |
| Interface web d'une org | `web:dgs:{deviceGroupSetId}` |
| Interface web d'un étab. | `web:dg:{deviceGroupId}` |
| Interface web d'une company | `web:company:{companyId}` |
| Interface admin | `web:admin` |

```
Source : inversive.common.web/Inversive.Common.Web/Realtime/Hubs/Constants/RealtimeConstants.cs:43
```

---

## RM-072 — Méthodes temps réel supportées

| Méthode | Description |
|---------|-------------|
| `SendACK` | Acquittement d'une commande |
| `SendStatus` | Mise à jour du statut du casque |
| `ManageExperience` | Lancer/arrêter une expérience |
| `ManageCast` | Démarrer/arrêter le casting |
| `ManageKiosk` | Activer/désactiver le mode kiosque |
| `GetLogs` | Récupérer les logs du casque |
| `Signal` | Signal générique de communication |

```
Source : inversive.common.web/Inversive.Common.Web/Realtime/Hubs/Constants/RealtimeConstants.cs:22
```
