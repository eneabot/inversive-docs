# Headset Center `[v4.0]`

Application **WPF desktop** (.NET 10.0-windows) utilisée pour configurer les casques autonomes via USB/ADB et gérer la flotte en temps réel.

---

## Contraintes générales

### RM-073 — Instance unique obligatoire

Une seule instance du Headset Center peut s'exécuter simultanément.  
Un **mutex nommé** `VRCXP.Configurator` l'enforce. Toute seconde instance est bloquée.

```
Source : headset-center/HeadsetCenter/Program.cs:271
```

---

### RM-074 — Détection USB par VID/PID matériel

| Constructeur | VID |
|---|---|
| Meta/Oculus | `0x2833` |
| Pico | `0x2D40` |
| HTC | `0x0BB4` ou `0x18D1` |

Les PIDs spécifiques distinguent les modes (développeur, MTP, ADB).

```
Source : headset-center/HeadsetCenter/UsbDetection/Enums.cs:17
```

---

## Processus de configuration en 12 étapes

### RM-075

| Étape | Nom | Description |
|-------|-----|-------------|
| 1 | `ChooseTargetType` | Sélection du modèle de casque |
| 2 | `ChooseStructure` | Sélection company / organisation / établissement (filtrée par rôle) |
| 3 | `ChooseConfiguration` | Choix ou création d'un profil de casque |
| 4 | `ChooseParameters` | Configuration kiosk mode, mot de passe, Wi-Fi |
| 5 | `ChooseExperiences` | Sélection des expériences à déployer |
| 6 | `ChooseSummary` | Récapitulatif avant déploiement |
| 7 | `PlugHeadset` | Connexion physique USB avec ADB activé |
| 8 | `DownloadApps` | Téléchargement des binaires (service, lobby, expériences) |
| 9 | `DeployApps` | Installation via ADB |
| 10 | `ErrorReport` | Affichage des erreurs éventuelles |
| 11 | `NamingHeadset` | Saisie du nom du casque |
| 12 | `SuccessScreen` | Configuration terminée |

```
Source : headset-center/HeadsetCenter/Model/StepType.cs
```

---

### RM-076 — Mot de passe kiosk obligatoire si kiosk mode activé

Si le kiosk mode est activé lors de la configuration, `KioskModePassword` est **obligatoire**.  
Le bouton « Continuer » reste désactivé tant que le champ est vide.

---

### RM-077 — 2 chemins de déploiement

| Chemin | Endpoint | Détail |
|--------|----------|--------|
| **LobbyCore** | `POST /standalone` | Déploie BundleAssets, ThirdPartyApps, CustomLobby. Config via `config.json` poussé sur le casque. |
| **Standard** | `POST /standalone/create` | Pousse les payloads d'expériences et données de device via intent Android `com.vrcxp.service.entry.CONFIGURATION`. |

---

### RM-078 — Noms de packages système fixes

| Composant | Package Android |
|-----------|----------------|
| Service | `com.vrcxp.service` |
| Lobby | `com.vrcxp.home` |
| Video Player | `com.Inversive.VideoPlayer` |

---

### RM-079 — Seuls fr-FR et en-US poussés sur le casque

Lors de la configuration, seules les langues `fr-FR` et `en-US` sont transmises au casque (filtrées depuis l'API `/langs`).

---

### RM-080 — Retry de téléchargement avec validation MD5

- Validation MD5 après chaque téléchargement
- En cas d'échec : le téléchargement est relancé
- Paramètres configurables via `appsettings.json` :
  - `DownloadRetry.MaxRetries`
  - `RetryDelayMs`
  - `TimeoutMs`

---

## Gestion de la flotte

### RM-081 — Tri des appareils par priorité de statut

| Priorité | Statut |
|----------|--------|
| 1 (premier) | `InGame` — en cours d'expérience |
| 2 | `TurnedOn` — allumé |
| 3 | `Standby` — veille |
| 4 | `TurnedOff` — éteint |
| 5 (dernier) | `Disconnected` — déconnecté |

À égalité de statut, tri par **ordre alphabétique**.

---

### RM-082 — Conditions des actions groupées

| Action | Condition de disponibilité |
|--------|---------------------------|
| Lancer une expérience | Au moins 1 appareil `TurnedOn` |
| Arrêter une expérience | Au moins 1 appareil `InGame` ou avec un `CurrentExperienceName` |
| Kiosk Mode | **Tous** les appareils sélectionnés doivent être `TurnedOn` |

---

### RM-083 — Expériences de test triées en dernier

Dans l'étape de sélection des expériences à déployer, les expériences marquées `isTesting` sont systématiquement placées **en fin de liste**.
