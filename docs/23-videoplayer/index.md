# VideoPlayer Android `[v4.0]`

Application **Unity Android** déployée sur les casques VR autonomes pour la lecture de vidéos 360°.  
Aucune communication API avec la plateforme — **fonctionnement entièrement local**.

---

## RM-137 — Paramètres de lancement via Intent Android

Le VideoPlayer reçoit sa configuration au démarrage via l'Intent Android, clé `videoPlayerData` (JSON) :

| Champ | Type | Description |
|-------|------|-------------|
| `experienceId` | `int` | Identifiant de l'expérience |
| `angle` | `string` | Format de projection vidéo |

Ces paramètres sont fournis par le **service Android** lors du lancement.

---

## RM-138 — Chemin de stockage des vidéos

```
/storage/emulated/0/Download/video360/{experienceId}/{experienceId}.mp4
```

Ce chemin est construit par `StaticMethods.GetGamePath(experienceId)` et doit être respecté par tout composant qui dépose les vidéos.

---

## RM-139 — Fichiers audio multilingues

Pattern : `{experienceId}_LANGUE.{extension}`

| Extension | Type |
|-----------|------|
| `.amb` | Audio ambisonic spatial |
| `.mp3` | MP3 |
| `.wav` | WAV |
| `.flac` | FLAC |
| `.tbe` | TBE |

Un audio par défaut intégré à la vidéo est toujours disponible.

---

## RM-140 — Fichiers de sous-titres

Pattern : `{experienceId}_LANGUE.srt`

**Seul le format SRT est supporté.**  
L'absence de fichier `.srt` signifie qu'aucun sous-titre n'est disponible.

---

## RM-141 — 6 formats de projection vidéo 360°

| Format | ID interne | Rendu |
|--------|-----------|-------|
| 180° | `2` | Mono |
| 360° | `3` | Mono |
| 360° Haut-Bas | `3` | Stéréo |
| 360° Gauche-Droite | `5` | Stéréo |
| 180° Haut-Bas | `11` | Stéréo |
| 180° Gauche-Droite | `13` | Stéréo |

Les formats **stéréo** activent le rendu binoculaire (deux yeux).  
Les formats **mono** n'utilisent qu'un seul rendu.

---

## RM-142 — Audio `.amb` active le mode spatial automatiquement

| Extension | `spatialize` Unity |
|-----------|-------------------|
| `.amb` | `true` (son spatial 3D) |
| Autres formats | `false` |
