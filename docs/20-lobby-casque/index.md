# Lobby casque `[v4.0]`

Environnement **3D Unity** tournant sur les casques VR autonomes.  
Point d'entrée de l'utilisateur sur le casque : authentification, catalogue d'expériences, lancement, feedback, paramètres.

---

## Authentification sur casque

### RM-104 — OAuth 2.0 Device Code

Le lobby déclenche le flux OAuth 2.0 Device Code via `IPCBridge.Login()`.  
Un `userCode` est reçu et affiché à l'utilisateur pour qu'il le saisisse sur la plateforme web.  
L'authentification est confirmée par callback `OnLoginSuccess`.

---

### RM-105 — Connexion invité : email obligatoire

Pour se connecter en tant qu'invité :

- L'utilisateur doit saisir une **adresse email**
- L'email ne peut pas être vide ou composer uniquement d'espaces (`string.IsNullOrWhiteSpace`)
- Si la validation échoue → l'action est silencieusement abandonnée

---

## Catalogue d'expériences

### RM-106 — Seules les expériences installées ou à installer sont affichées

Le catalogue du lobby n'affiche que les expériences répondant à :

- `isInstalled == true` — APK installé sur le casque, **ou**
- `isInstallRequired == true` — installation requise pour accès

Les expériences non installées et non requises sont **ignorées**.

---

### RM-107 — Suppression des expériences disparues lors du refresh

Lors d'un refresh périodique, les expériences qui n'apparaissent plus dans la réponse du service sont **automatiquement retirées** de l'affichage.

---

## Lancement d'expériences

### RM-108 — Uptale : utilisateur authentifié obligatoire

Les expériences **Uptale** ne peuvent être lancées que si un utilisateur est authentifié (`CurrentUserProfile != null`).  
Les accès invité (par invitation) sont **insuffisants**.

---

### RM-109 — Onglet feedback visible seulement si `isTesting = true`

L'onglet de feedback (like/dislike + commentaire + captures d'écran) n'est visible que si `isTesting == true`.  
Pour les expériences validées (statut Actif), cet onglet **n'est pas affiché**.

---

## Paramètres & Comportement

### RM-110 — Composants UI filtrés par modèle de casque

Certains composants de l'interface peuvent être activés ou désactivés selon le `DeviceStandaloneType`.  
Ce mécanisme (`ActiveOnDeviceType`) permet d'afficher ou masquer des fonctionnalités spécifiques à un constructeur.

---

### RM-111 — Volume et luminosité : plage 0–100

Volume audio et luminosité configurables de **0 à 100**.

Conversion volume :
```
volumeAbsolu = (volume × volumeMax) / 100
```

---

### RM-112 — Lobby signale « prêt » après chargement des assets

Le lobby envoie `OnLobbyReady()` au service Android **uniquement après** que les assets (customisation, langue, logo) ont été chargés avec succès.  
Ce signal déclenche les opérations post-initialisation côté service.

---

### RM-113 — IPC fire-and-forget

La communication entre le lobby (Unity) et le service Android passe par un bridge JNI (`IPCFacade`).

- Chaque appel est **fire-and-forget** — le lobby ne bloque pas en attendant la réponse
- Les résultats arrivent via callbacks `OnMethodSuccess` / `OnMethodFailure`
- Les erreurs critiques déclenchent automatiquement un **popup d'erreur**
