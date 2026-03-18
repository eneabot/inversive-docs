# Compatibilité des appareils

## Matrice de compatibilité

| Appareil | WebGL | VR filaire | APK AR | Casque sans fil |
|----------|-------|------------|--------|-----------------|
| **iPhone/iPad** | ✅ (pas de plein écran) | ❌ | ❌ | ❌ |
| **Android (smartphone)** | ✅ | ❌ | ✅ | ❌ |
| **PC Windows** | ✅ | ✅ (SteamVR) | ❌ | ❌ |
| **Casques autonomes** | ❌ | ❌ | ❌ | ✅ |

---

## RM-057 — Smartphones Apple : WebGL uniquement, pas de plein écran

Les smartphones Apple ne peuvent consommer que les expériences de type **Navigateur (WebGL)**.  
Le contenu WebGL **ne peut pas être affiché en plein écran** sur iOS.

---

## RM-058 — Smartphones Android

Les smartphones Android peuvent consommer :

- Les expériences **navigateur (WebGL)**
- Les expériences de **Réalité Augmentée** (téléchargement APK)

---

## RM-059 — PC Windows

Un PC Windows peut consommer :

- Les expériences **WebGL** via navigateur
- Les expériences **VR filaires** avec un casque SteamVR + **Immersive Catalog**

---

## RM-060 — Mise à jour casque nécessite internet

Si des informations ou paramètres d'un casque ont été modifiés, le casque doit être **connecté à internet** pour que les mises à jour soient appliquées.

---

## RM-060b — Casques VR sans fil supportés

| Constructeur | Modèles |
|---|---|
| **Meta** | Quest 2, Quest 3, Quest 3S |
| **Pico** | G3, 4 Enterprise, 4 Ultra Enterprise, Neo 3 Pro, Neo 3 Pro Eye |
| **HTC** | Vive Focus Vision, Vive XR Elite |

---

## RM-060c — Disparité fonctionnelle entre constructeurs

Les fonctionnalités disponibles peuvent **varier selon le constructeur**, en fonction de la capacité d'intégration que chaque constructeur met à disposition.

---

## RM-060d — Expériences WebGL : 3 formats de compression Unity supportés

| Format | Compression |
|--------|-------------|
| `br` | Brotli |
| `gz` | gzip |
| *(aucun)* | Sans compression |
