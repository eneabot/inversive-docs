# Intégration LTI — `keycloak-lti-mapper` `[v4.0]`

Extension **Keycloak** (Java 17, Keycloak 26.0.0) qui enrichit les tokens OIDC avec les claims LTI 1.3 nécessaires à l'intégration Uptale.  
Déployée comme mapper Keycloak sur les clients LTI.

---

## Flux de lancement LTI

### RM-095a — Version LTI fixée à 1.3.0

Le mapper supporte exclusivement **LTI 1.3.0**.  
La version est systématiquement définie à `"1.3.0"` dans le token JWT généré.

---

### RM-095b — Détection automatique du type de message

| URL cible contient | `message_type` |
|---|---|
| `/lti/deeplinkSelect` | `LtiDeepLinkingRequest` |
| Autre | `LtiResourceLinkRequest` |

---

### RM-095c — Hiérarchie de résolution de l'URI cible

Ordre de priorité :

1. `lti_message_hint` décodé en Base64 *(source primaire)*
2. Paramètre URL `target_link_uri` *(flux standard)*
3. URI par défaut configuré (`default.target.uri`, défaut : `https://mydev.uptale.io/dashboard`)

---

### RM-095d — `lti_message_hint` encodé en Base64

Le `lti_message_hint` est un objet **JSON encodé en Base64**.  
Champs possibles : `target_link_uri`, `deep_link_return_url`, `data`, `device_groups`

Recherche dans l'ordre : AuthenticationSession notes → ClientSession notes → paramètres URL

---

## Mappage des rôles

| Groupe Keycloak | Rôle LTI attribué |
|---|---|
| `Uptale_Creator` | Instructor |
| Contient `admin` ou `owner` | Instructor **+** Administrator |
| Aucun des précédents | **Learner** (défaut) |

La comparaison des noms de groupe est **insensible à la casse**.

---

## Claims générées

### RM-095h — ResourceLink : UUID aléatoire

Pour `LtiResourceLinkRequest` : la claim `resource_link` contient un **UUID aléatoire** et le titre fixe `"Inversive Experience"`.  
Absente des `LtiDeepLinkingRequest`.

---

### RM-095i — Deep linking : `accept_multiple = true`

Pour `LtiDeepLinkingRequest` :

- `accept_multiple = true` obligatoire
- Types acceptés : `link`, `file`, `html`, `ltiResourceLink`, `image`
- Cibles : `iframe`, `window`, `embed`
- Le champ `data` contient le ticket ID

---

### RM-095j — Claim `context` : requiert `company_id` et `company_name`

La claim `context` n'est générée que si les attributs Keycloak `company_id` et `company_name` sont renseignés.  
Ces attributs doivent être synchronisés depuis le backend Inversive.

---

### RM-095k — Claim `inversive_device_groups`

Priorité de résolution : `lti_message_hint.device_groups` → attribut utilisateur Keycloak `device_groups` → liste vide

---

### RM-095l — Claims injectées dans 3 types de tokens

Les claims LTI sont présentes dans :

- **ID Token**
- **Access Token**
- **UserInfo Token**

---

## Configuration

### RM-095m — Deployment ID

Configurable via `deployment.id`.  
Valeur par défaut : `inversive-uptale-1`

### RM-095n — URL de callback deep linking

Résolution : `lti_message_hint` → propriété mapper `platform.callback.url`  
Si aucune source ne la fournit → avertissement critique journalisé, mais le processus continue.
