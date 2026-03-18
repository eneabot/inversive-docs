# Notes de conformité

Cette annexe documente les **écarts identifiés entre le document v2.0 et le code source**.  
Ces écarts ne représentent pas des bugs mais des divergences sémantiques ou des détails d'implémentation à connaître.

---

## CONF-001 — Politique de mot de passe (→ RM-002)

| | Détail |
|-|--------|
| **Document v2.0** | Caractères spéciaux autorisés : `` $@&()[]{}=#,-!?+/£€% `` |
| **Code** | Aucune validation de pattern au niveau des modèles API. Le champ `Password` est marqué `[Required]` uniquement. |
| **Explication** | La politique de mot de passe est appliquée par **Keycloak** (Identity Provider), pas par l'API Inversive. |
| **Action recommandée** | Vérifier la configuration Keycloak pour s'assurer qu'elle correspond aux règles documentées. |

---

## CONF-002 — Validité d'une invitation de test (→ RM-017)

| | Détail |
|-|--------|
| **Document v2.0** | « L'invitation se fait par email avec un lien d'accès valable 15 jours par défaut. » |
| **Code** | `invitation.Expiration < DateTime.Today` — le **jour d'expiration est inclus** jusqu'en fin de journée (23:59:59). |
| **Explication** | La comparaison utilise `DateTime.Today` (date sans heure). Si l'expiration est fixée à J+15, l'invitation reste valide jusqu'à la fin de la journée J+15. |
| **Action recommandée** | Mettre à jour RM-017 : « valable 15 jours par défaut, expiration incluse jusqu'en fin de journée. » |

---

## CONF-003 — Mode Kiosque et Pico (→ RM-036)

| | Détail |
|-|--------|
| **Document v2.0** | « Les actions Mode Kiosk, Éteindre et Redémarrer sont disponibles uniquement pour les casques Pico Enterprise. » |
| **Code** | Le champ `KioskMode` est présent sur l'entité `DeviceStandalone` pour **tous les modèles de casques**, sans filtrage par constructeur dans le modèle de données. |
| **Explication** | La restriction Pico est probablement appliquée au niveau de l'**interface utilisateur** (UI) ou au niveau des commandes envoyées au casque. |
| **Action recommandée** | Confirmer si la restriction est côté API (à implémenter) ou purement côté UI (à documenter ainsi). |

---

## CONF-004 — Terminologie métier vs technique

| Terme métier (document v2.0) | Terme technique (code) |
|------------------------------|------------------------|
| Gestionnaire d'organisation | `DeviceGroupSetManager` |
| Gestionnaire d'établissement | `DeviceGroupManager` |
| Organisation | `DeviceGroupSet` |
| Établissement | `DeviceGroup` |
| Apprenant | `User` (BaseUserType) |
| Producteur de contenu | `Producer` (BaseUserType) |
| Administrateur | `CompanyOwner` (BaseUserType) |
| Super Admin | `Admin` (BaseUserType) |

Ces deux vocabulaires coexistent dans la plateforme.  
Le document v2.0 utilise le vocabulaire **métier** ; le code utilise la terminologie **technique**.  
Les deux sont valides dans leurs contextes respectifs.
