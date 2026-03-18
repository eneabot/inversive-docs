# Actions sur casques

## RM-036 — Mode Kiosk, Éteindre et Redémarrer : Pico uniquement

Les actions suivantes sont disponibles **uniquement pour les casques Pico Enterprise** :

- Activer/Désactiver le **Mode Kiosk**
- **Éteindre** un casque à distance
- **Redémarrer** un casque à distance

!!! warning "Note de conformité — CONF-003"
    Dans le modèle de données, `KioskMode` est présent sur **tous** les `DeviceStandalone` sans filtrage par constructeur.  
    La restriction Pico est appliquée au niveau de l'**interface utilisateur**, pas du modèle de données.

!!! info
    L'activation/désactivation du Mode Kiosk entraîne un **redémarrage** du casque.

---

## RM-037 — Actions à distance : même réseau Wi-Fi requis

Pour que les actions à distance soient disponibles (casting, lancement d'expérience, mode kiosk…), le **Headset Center et les casques doivent être sur le même réseau Wi-Fi**.

---

## RM-038 — Mot de passe kiosk

Le mot de passe permettant de désactiver ou réactiver le Mode Kiosk est connu **uniquement des utilisateurs ayant accès aux profils de casques**.

---

## RM-038b — Casting = vue en direct

L'action **Casting** (mise en miroir) permet de voir en temps réel ce que l'utilisateur voit dans le casque.  
Le **multicasting** permet de visualiser plusieurs flux de casques simultanément.

---

## RM-038c — Actions groupées : bouton actif dès 1 casque sélectionné

Le bouton « Actions » sur une sélection multiple de casques devient **actif dès qu'au moins un casque est sélectionné**.  
Les actions s'affichent pour tous les casques sélectionnés, même si certaines ne sont effectives que sur certains modèles.

---

## RM-038d — Export des données de la flotte

Il est possible d'**exporter les données de la flotte** de casques depuis l'interface web (page Appareils).
