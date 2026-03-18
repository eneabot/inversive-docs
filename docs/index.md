# INVERSIVE DATA BOOK

<div class="databook-hero">
<h2>INVERSIVE DATA BOOK</h2>
<p>v4.0 · Mars 2026 · Source of Truth — Règles Métier de la Plateforme Inversive</p>
<div class="stats">
  <div class="stat">
    <div class="stat-value">320+</div>
    <div class="stat-label">Règles</div>
  </div>
  <div class="stat">
    <div class="stat-value">24</div>
    <div class="stat-label">Modules</div>
  </div>
  <div class="stat">
    <div class="stat-value">8</div>
    <div class="stat-label">Composants</div>
  </div>
  <div class="stat">
    <div class="stat-value">v4.0</div>
    <div class="stat-label">Version</div>
  </div>
</div>
</div>

Ce document est la **source de vérité** pour toutes les règles métier de la plateforme Inversive.
Destiné aux développeurs, product owners, QA et toute personne qui travaille sur le produit.

---

## Index des modules

<div class="databook-grid">

<a href="01-comptes/" class="databook-card cat-auth">
<div class="databook-card-id">MODULE #01</div>
<div class="databook-card-title">🔐 Authentification & Comptes</div>
<div class="databook-card-desc">Création de compte, activation, connexion, JWT, SSO, LTI, politique de mot de passe.</div>
<div class="databook-card-count">RM-001 → RM-007</div>
</a>

<a href="02-utilisateurs/" class="databook-card cat-users">
<div class="databook-card-id">MODULE #02</div>
<div class="databook-card-title">👥 Utilisateurs & Rôles</div>
<div class="databook-card-desc">6 rôles hiérarchiques, droits d'accès, usurpation d'identité, statuts, rôles personnalisés.</div>
<div class="databook-card-count">RM-008 → RM-013</div>
</a>

<a href="03-structure/" class="databook-card cat-struct">
<div class="databook-card-id">MODULE #03</div>
<div class="databook-card-title">🏗️ Structure Organisationnelle</div>
<div class="databook-card-desc">Hiérarchie Company → Organisation → Établissement, modules, archivage.</div>
<div class="databook-card-count">RM-014 → RM-014e</div>
</a>

<a href="04-experiences/" class="databook-card cat-exp">
<div class="databook-card-id">MODULE #04</div>
<div class="databook-card-title">🎮 Expériences</div>
<div class="databook-card-desc">Création, validation, binaires, catalogue, disponibilité, invitations de test.</div>
<div class="databook-card-count">RM-015 → RM-037</div>
</a>

<a href="05-lancement/" class="databook-card cat-launch">
<div class="databook-card-id">MODULE #05</div>
<div class="databook-card-title">🚀 Lancement d'Expériences</div>
<div class="databook-card-desc">Modes de lancement, catalogue WebGL, Immersive Catalog, sessions.</div>
<div class="databook-card-count">RM-038 → RM-038c</div>
</a>

<a href="06-casques/" class="databook-card cat-headset">
<div class="databook-card-id">MODULE #06</div>
<div class="databook-card-title">🥽 Casques Autonomes</div>
<div class="databook-card-desc">Configuration, profils, actions en temps réel, télémétrie, réinitialisation.</div>
<div class="databook-card-count">RM-038d → RM-038j</div>
</a>

<a href="07-statistiques/" class="databook-card cat-stats">
<div class="databook-card-id">MODULE #07</div>
<div class="databook-card-title">📊 Statistiques</div>
<div class="databook-card-desc">Périmètre par rôle, filtres, export Excel, 3 onglets de données.</div>
<div class="databook-card-count">RM-039 → RM-041c</div>
</a>

<a href="08-assets/" class="databook-card cat-assets">
<div class="databook-card-id">MODULE #08</div>
<div class="databook-card-title">📦 Bibliothèque d'Assets</div>
<div class="databook-card-desc">Statuts, formats acceptés, import groupé, BundleAssets, IsVanilla.</div>
<div class="databook-card-count">RM-042 → RM-046h</div>
</a>

<a href="09-guides/" class="databook-card cat-auth">
<div class="databook-card-id">MODULE #09</div>
<div class="databook-card-title">📖 Guides d'Utilisation</div>
<div class="databook-card-desc">Accès, formats PDF, commentaires, documentation personnalisée.</div>
<div class="databook-card-count">RM-047 → RM-048d</div>
</a>

<a href="10-sdk/" class="databook-card cat-sdk">
<div class="databook-card-id">MODULE #10</div>
<div class="databook-card-title">⚙️ SDK Inversive</div>
<div class="databook-card-desc">Chapitrage Unity, scoring, sessions, reporting, stockage local, déploiement.</div>
<div class="databook-card-count">RM-049 → RM-053l</div>
</a>

<a href="11-personnalisation/" class="databook-card cat-struct">
<div class="databook-card-id">MODULE #11</div>
<div class="databook-card-title">🎨 Personnalisation</div>
<div class="databook-card-desc">Marque blanche, 13 options visuelles, salon virtuel, personnalisation par niveau.</div>
<div class="databook-card-count">RM-054 → RM-056d</div>
</a>

<a href="12-compatibilite/" class="databook-card cat-headset">
<div class="databook-card-id">MODULE #12</div>
<div class="databook-card-title">📱 Compatibilité Appareils</div>
<div class="databook-card-desc">Matrice iOS/Android/PC/VR, modèles supportés, disparités constructeurs.</div>
<div class="databook-card-count">RM-057 → RM-060d</div>
</a>

<a href="13-evenements/" class="databook-card cat-launch">
<div class="databook-card-id">MODULE #13</div>
<div class="databook-card-title">🎪 Événements Publics</div>
<div class="databook-card-desc">Accès sans auth, PublicEventId, clôture automatique, archivage de sessions.</div>
<div class="databook-card-count">RM-061 → RM-064</div>
</a>

<a href="14-lobby/" class="databook-card cat-exp">
<div class="databook-card-id">MODULE #14</div>
<div class="databook-card-title">🏛️ Lobby Personnalisé</div>
<div class="databook-card-desc">CustomLobby, 4 niveaux d'assignation, BundleAssets, ThirdPartyApps.</div>
<div class="databook-card-count">RM-065 → RM-066</div>
</a>

<a href="15-localisation/" class="databook-card cat-users">
<div class="databook-card-id">MODULE #15</div>
<div class="databook-card-title">🌍 Localisation</div>
<div class="databook-card-desc">7 langues d'interface, 100+ langues de contenu, traductions par company.</div>
<div class="databook-card-count">RM-067 → RM-069</div>
</a>

<a href="16-temps-reel/" class="databook-card cat-sdk">
<div class="databook-card-id">MODULE #16</div>
<div class="databook-card-title">⚡ Temps Réel & SignalR</div>
<div class="databook-card-desc">Redis backplane, clés scopées multi-tenant, méthodes hub, authentification.</div>
<div class="databook-card-count">RM-070 → RM-072</div>
</a>

<a href="17-headset-center/" class="databook-card cat-app">
<div class="databook-card-id">MODULE #17 — APP</div>
<div class="databook-card-title">🖥️ Headset Center</div>
<div class="databook-card-desc">Config casques via USB/ADB, 12 étapes de déploiement, gestion de flotte.</div>
<div class="databook-card-count">RM-073 → RM-083</div>
</a>

<a href="18-immersive-catalog/" class="databook-card cat-app">
<div class="databook-card-id">MODULE #18 — APP</div>
<div class="databook-card-title">📂 Immersive Catalog</div>
<div class="databook-card-desc">App desktop VR, téléchargement/installation, protocole vrcxp://, systray.</div>
<div class="databook-card-count">RM-084 → RM-093</div>
</a>

<a href="19-service-android/" class="databook-card cat-headset">
<div class="databook-card-id">MODULE #19 — ANDROID</div>
<div class="databook-card-title">🤖 Service Android</div>
<div class="databook-card-desc">Sync expériences, 5 actions, auth OAuth Device Code, logs, auto-update.</div>
<div class="databook-card-count">RM-094 → RM-103</div>
</a>

<a href="20-lobby-casque/" class="databook-card cat-exp">
<div class="databook-card-id">MODULE #20 — UNITY</div>
<div class="databook-card-title">🎯 Lobby Casque</div>
<div class="databook-card-desc">Environnement Unity VR, auth Device Code, catalogue, IPC fire-and-forget.</div>
<div class="databook-card-count">RM-104 → RM-113</div>
</a>

<a href="21-lti/" class="databook-card cat-auth">
<div class="databook-card-id">MODULE #21 — LTI</div>
<div class="databook-card-title">🎓 Intégration LTI</div>
<div class="databook-card-desc">Keycloak mapper LTI 1.3, claims OIDC, rôles Uptale, deep linking.</div>
<div class="databook-card-count">RM-095a → RM-095n</div>
</a>

<a href="22-api-standalone/" class="databook-card cat-sdk">
<div class="databook-card-id">MODULE #22 — API</div>
<div class="databook-card-title">🔌 API Standalone</div>
<div class="databook-card-desc">Auth casques, JWT, rotation refresh tokens, URLs présignées MinIO, SignalR.</div>
<div class="databook-card-count">RM-114 → RM-136</div>
</a>

<a href="23-videoplayer/" class="databook-card cat-exp">
<div class="databook-card-id">MODULE #23 — UNITY</div>
<div class="databook-card-title">🎬 VideoPlayer 360°</div>
<div class="databook-card-desc">6 formats de projection, audio spatial, sous-titres SRT, Intent Android.</div>
<div class="databook-card-count">RM-137 → RM-142</div>
</a>

<a href="24-blazor/" class="databook-card cat-app">
<div class="databook-card-id">MODULE #24 — FRONTEND</div>
<div class="databook-card-title">🌐 Application Web Blazor</div>
<div class="databook-card-desc">Tokens, auth, validations, routing, SignalR client, localStorage, recherche.</div>
<div class="databook-card-count">RM-143 → RM-177</div>
</a>

</div>

---

## Comment lire ce document

Chaque règle est identifiée par un code **RM-XXX** (Rule Métier).

| Mention | Signification |
|---------|--------------|
| *(aucune)* | Règle du Data Book v2.0 (historique) |
| `[v3.0]` | Extraite du code source en v3.0 |
| `[v4.0]` | Ajoutée ou précisée en v4.0 |

---

## Correspondance terminologique rapide

| Terme métier | Terme technique | `BaseUserType` |
|---|---|---|
| Super Admin | `Admin` | `1` |
| Administrateur | `CompanyOwner` | `2` |
| Gestionnaire d'organisation | `DeviceGroupSetManager` | `4` |
| Gestionnaire d'établissement | `DeviceGroupManager` | `8` |
| Apprenant | `User` | `16` |
| Producteur de contenu | `Producer` | `32` |

→ [Table complète](annexes/terminologie.md)
