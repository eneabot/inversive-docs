# Immersive Catalog `[v4.0]`

Application **WPF desktop** (.NET) permettant aux utilisateurs de naviguer dans le catalogue d'expériences, télécharger les binaires et lancer des expériences VR filaires ou standard sur PC.

---

## Contraintes générales

### RM-084 — Instance unique et récupération après crash

- Une seule instance autorisée (fermeture immédiate si doublon détecté)
- En cas de crash : redémarrage automatique avec l'argument `-fromcrash`, délai de **1 seconde** avant démarrage

---

### RM-085 — Mode hors ligne conditionnel

Le mode hors ligne est disponible **uniquement** si un token valide est présent en cache local.  
Sans token → erreur de connexion.  
En mode hors ligne, le dossier du catalogue doit aussi exister localement.

---

### RM-086 — Timeout HTTP de 72 heures pour les téléchargements

Le client HTTP pour les téléchargements a un timeout de **72 heures**, nécessaire pour les fichiers volumineux sur connexions lentes.

---

## Téléchargement & Installation

### RM-087 — 4 comportements selon le type d'expérience

| Type | Traitement |
|------|-----------|
| **VR** | Téléchargé en ZIP, extrait dans `Binaries/`, point d'entrée requis |
| **Video360** | Téléchargé comme `Binaries.mp4`, pas d'extraction |
| **ExternalApplication** | Téléchargé dans `Binaries/{EntryPoint}`, pas d'extraction |
| **Autres** (WebGL, Standard…) | Téléchargés en ZIP, extraits dans `Binaries/` |

Les binaires externes (sous-titres, audio) des Video360 sont téléchargés séparément depuis MinIO.

---

### RM-088 — Validation MD5 + retry max 3 tentatives

Après chaque téléchargement :

1. Hash MD5 calculé et comparé au hash fourni par l'API
2. En cas de divergence : retéléchargement avec **1 seconde de délai**
3. Maximum : **3 tentatives**

!!! info "Exemptions"
    Video360 et ExternalApplication sont **exemptés** de la validation MD5.

---

### RM-089 — Détection de mise à jour

| Champ | Déclenche |
|-------|-----------|
| `LastChangeDate` | Re-téléchargement des **assets** |
| `BuildLastUploadDate` | Re-téléchargement des **binaires** |

---

### RM-090 — Répertoires obsolètes supprimés automatiquement

Lors de la synchronisation avec l'API, les dossiers locaux d'expériences qui n'existent plus dans la réponse API sont **automatiquement supprimés**, ainsi que leurs entrées dans `Settings.ExperienceStatuses`.

---

### RM-091 — Écriture JSON atomique avec 10 tentatives max

Les fichiers `experience.json` sont écrits de manière **atomique** :

1. Écriture dans un fichier temporaire
2. Remplacement par `File.Replace()`
3. En cas d'échec (fichier verrouillé) : jusqu'à **10 tentatives** avant abandon

---

## Lancement & Intégration

### RM-092 — Protocole URL `vrcxp://`

L'Immersive Catalog est enregistré comme gestionnaire du protocole URL `vrcxp://`.

**Flux :**
1. URL `vrcxp://` écrit l'ID de l'expérience dans le registre Windows (`HKEY_CURRENT_USER\Software\Inversive\VRC`)
2. Un watcher de registre détecte le changement
3. Le catalogue se lance avec l'expérience cible

---

### RM-093 — Clic sur le X = masqué (pas fermé)

L'Immersive Catalog se comporte comme une application **systray**.  
Cliquer sur le bouton de fermeture **masque la fenêtre** sans quitter l'application.  
Une notification balloon s'affiche pendant **2 secondes**.
