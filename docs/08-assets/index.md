# Bibliothèque d'assets

## Statuts d'un asset

| Statut | Description |
|--------|-------------|
| **Publié** | Visible dans la bibliothèque |
| **Dans le panier** | Sélectionné par l'utilisateur |
| **En cours de demande d'accès** | En attente de validation admin |
| **Disponible au téléchargement** | Accès accordé par l'admin |

---

## RM-042 — 4 statuts pour un asset

Voir tableau ci-dessus.

---

## RM-043 — Demande d'accès validée par l'administrateur

L'accès à un asset est accordé ou refusé par l'**administrateur de l'entreprise** après réception d'une notification email.  
Le statut est mis à jour en **temps réel**.

---

## RM-044 — Formats acceptés par type d'asset

| Type | Formats acceptés |
|------|-----------------|
| **3D** | `.obj`, `.glb`, `.gltf`, `.fbx` |
| **2D** | `.jpg`, `.jpeg`, `.png` |
| **VFX** | `.psd`, `.tga`, `.tiff`, `.dpx` |
| **UnityPackage** | — |
| **Vidéo** | `.mp4` |
| **Audio** | `.mp3`, `.wav`, `.m4a` |

!!! info
    Tous les fichiers doivent être fournis dans un dossier compressé **`.zip`**.

---

## RM-045 — Jusqu'à 4 formats simultanés pour un asset 3D

Pour un même asset 3D, il est possible de téléverser **jusqu'à 4 formats simultanément** (`.obj`, `.glb`, `.gltf`, `.fbx`).

---

## RM-046 — Publication notifie les administrateurs

Lors de la publication d'un nouvel asset, **tous les administrateurs** de l'entreprise concernée reçoivent un email de notification.

---

## RM-046b — Import groupé via .zip structuré

Il est possible d'importer plusieurs assets via un **fichier .zip respectant une architecture de dossiers** précise (téléchargeable comme modèle).  
Un fichier `.json` peut compléter les métadonnées.

---

## RM-046c — Prévisualisation avant import groupé

Avant de lancer l'import, une prévisualisation récapitule :

- Nombre total d'assets
- Nombre de dossiers valides
- Nombre de dossiers en erreur

Chaque asset peut être **modifié individuellement** avant l'import.

---

## RM-046d — Reprise possible si import interrompu

Si une erreur survient durant un import groupé, il est possible de **relancer l'import uniquement pour les assets dont l'import a été interrompu**.

---

## RM-046e — Types d'assets disponibles

Modèles 3D · Images 2D · VFX · Animations · Vidéos · Textures et matériaux · Audios

---

## Règles techniques `[v3.0]`

### RM-046f — BundleAsset IsVanilla : accessible à toutes les companies

Un `BundleAsset` avec `IsVanilla = true` est accessible à **toutes les companies** sans restriction.  
Pas besoin d'association `CompanyBundleAsset`.

### RM-046g — BundleAsset non-vanilla : accès restreint

Un `BundleAsset` avec `IsVanilla = false` n'est accessible que si la company possède une association `CompanyBundleAsset` correspondante.

### RM-046h — Types de contenus AssetBundle

| Valeur | Type | Description |
|--------|------|-------------|
| `0` | Theme/Décor | Ambiances visuelles |
| `1` | AdditionalContent | Contenus additionnels |
| `2` | Scene | Scènes 3D complètes |
