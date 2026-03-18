# Configuration & Déploiement `[v4.0]`

## RM-053j — SDK Production vs SDK Dev

Les deux packages Unity sont **fonctionnellement identiques**. Seule l'URL de base des appels API diffère :

| Package | Version | URL de base |
|---------|---------|------------|
| `inversive-sdk` | v1.1.2 | `https://sdk.vrcxp.com/` |
| `inversive-dev-sdk` | v1.1.6 | `https://dev-sdk.vrcxp.com/` |

```
Source : inversive-sdk/Runtime/InversiveUtilities.cs:8
         inversive-dev-sdk/Runtime/InversiveUtilities.cs:8
```

---

## RM-053k — 5 types de dispositifs supportés

Le SDK reconnaît 5 modes via `DeviceTypeEnum` :

| Valeur | Type |
|--------|------|
| `0` | Terminal |
| `1` | Personal |
| `2` | WallScreen |
| `3` | Standalone |
| `4` | WebBrowser |

Pour le type `Standalone`, un sous-type `DeviceStandaloneTypeEnum` précise le modèle :

| Valeur | Modèle |
|--------|--------|
| `0` | None |
| `1` | Meta Quest 2 |
| `2` | Pico 4 Enterprise |
| `3` | Meta Quest 3 |

```
Source : inversive-sdk/Runtime/InversiveClasses.cs:222-243
```
