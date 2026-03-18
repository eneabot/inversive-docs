# Service Android — `inversive_service` `[v4.0]`

Service Android tournant en **arrière-plan** sur les casques autonomes.  
Gère la synchronisation des expériences, le reporting de télémétrie, l'authentification, le mode kiosk et les auto-mises à jour.

---

## Synchronisation des expériences

### RM-094 — 5 actions de synchronisation

Lors de chaque sync, chaque expérience est comparée avec le cache local et reçoit l'une des 5 actions :

| Action | Condition | Traitement |
|--------|-----------|------------|
| **Replace** | Version changée ET nom de package différent | Désinstaller l'ancien + installer le nouveau |
| **Upgrade** | Version changée (`lastUpload` ou `lastUpdate`) | Mettre à jour APK et/ou assets |
| **Install** | `isInstallRequired = true`, ou expérience de test non installée | Installer |
| **Uninstall** | `isUninstallRequired = true` | Désinstaller |
| **None** | Aucun changement détecté | Aucune action |

---

### RM-095 — Critères de mise à jour APK vs assets

| Critère | Déclenche |
|---------|-----------|
| `remote.lastUpload > local.lastUpload` | Mise à jour **APK** |
| `remote.lastUpdate > local.lastUpdate` | Mise à jour **assets** |

Ces deux mises à jour sont **indépendantes** et peuvent se produire simultanément.

---

### RM-096 — Sync hors ligne : cache local uniquement

Si le Wi-Fi est déconnecté lors d'une sync :

- Le service charge les expériences depuis la **base de données locale**
- **Aucune mise à jour ni installation** n'est effectuée

---

### RM-097 — Auto-mises à jour système uniquement si Wi-Fi connecté

Les auto-mises à jour du service, du lobby, du lecteur vidéo et du lecteur Uptale ne sont déclenchées que si le **Wi-Fi est connecté**.

Ordre de mise à jour (séquentiel) :
```
service → lobby → lecteur vidéo → lecteur Uptale
```

---

## Gestion des sessions

### RM-098 — Détection de fermeture d'expérience via UsageStats

| Paramètre | Valeur |
|-----------|--------|
| Polling interval | **1 seconde** sur une fenêtre de 10 secondes |
| Debounce | **500 ms** avant de reporter la fermeture |
| Seuil d'inactivité | L'expérience doit être en arrière-plan depuis **≥ 3 secondes** |
| Overlays système | Ignorés |

---

## Authentification & Réseau

### RM-099 — Authentification via OAuth 2.0 Device Code

**Flux :**
1. Le service reçoit un `userCode` + `deviceCode` avec expiration et intervalle de polling
2. Le `userCode` est diffusé à l'interface lobby pour affichage
3. Le service interroge le serveur à l'intervalle donné jusqu'à obtention du token

---

### RM-100 — Timeouts réseau

| Type de requête | Timeout |
|---|---|
| Requêtes API (request/connect/socket) | 30 secondes |
| Téléchargements de fichiers | 15 minutes |
| Polling DeviceCode | 60 secondes max |

---

### RM-101 — Reconnexion HeadsetCenter : backoff exponentiel

| Paramètre | Valeur |
|-----------|--------|
| Délai initial | 1 seconde |
| Facteur multiplicateur | ×2 |
| Maximum | 30 secondes |
| Nombre max de tentatives | **2** |

---

### RM-102 — Uptale : token SSO Bearer requis

Le lancement d'une expérience **Uptale** requiert un **token Bearer de type SSO** (pas un JWT standard).  
Si le Bearer disponible n'est pas de type SSO, le lancement échoue.  
Le token SSO est passé à l'URL de deep linking : `https://app.uptale.io/deeplinking/ExternalLaunch`

---

## Logs

### RM-103 — Logs uploadés en ZIP avec nettoyage automatique

1. Logs compressés dans un fichier **ZIP horodaté**
2. Uploadés via l'endpoint `/device/data`
3. Le fichier ZIP local est **supprimé après upload**

Répertoire des logs : `context.filesDir/logs`
