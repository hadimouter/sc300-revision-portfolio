# Identités de workload

Domaine « Plan and implement workload identities », 20 à 25 % de l'examen.

Une identité de workload est une identité qui n'appartient pas à un humain : application, service, script ou machine. Les questions demandent surtout de choisir le bon type d'identité, le bon mécanisme d'authentification et le strict nécessaire côté autorisation.

## Application object et service principal

Enregistrer une application crée **deux objets distincts**.

L'**application object** est la définition globale : identifiants, URI de redirection, secrets et certificats, permissions demandées, API exposée, app roles. Il vit dans le tenant d'origine et apparaît sous App registrations.

Le **service principal** est l'instance locale de l'application dans un tenant. Il porte les affectations, consentements, app role assignments et propriétés propres à ce tenant. Il apparaît sous Enterprise applications.

Pour une application multi-tenant, il existe un application object chez l'éditeur et un service principal dans chaque tenant client où l'application est consentie.

Les deux partagent le même **Application (client) ID**, mais ont des **Object ID différents**.

| | App registrations | Enterprise applications |
|---|---|---|
| Objet | Application object | Service principal |
| Configuration | Authentication, redirect URIs, credentials, API permissions, Expose an API, App roles, Owners, Manifest | Assignment required, Users and groups, Single sign-on, Provisioning, consentements, Sign-in logs |
| Portée | Tenant d'origine | Tenant local |

Un service principal peut aussi exister sans application object local, notamment pour des applications Microsoft ou des managed identities.

## Permissions déléguées et permissions applicatives

| | Déléguée | Applicative |
|---|---|---|
| Contexte | Au nom d'un utilisateur connecté | Sans utilisateur, identité propre de l'application |
| Objet Entra | `oauth2PermissionScope` | `appRole` |
| Claim | `scp` | `roles` dans un token app-only |
| Consentement | utilisateur si autorisé, ou administrateur | administrateur |
| Portée effective | droits de l'application bornés par le contexte de l'utilisateur | permissions accordées à l'application |

Le nom seul ne suffit jamais : `User.Read.All` peut exister en délégué ou en applicatif.

### Le claim `roles` n'est pas réservé aux tokens app-only

Un token utilisateur peut lui aussi contenir `roles`, pour y porter des app roles métier.

Lecture correcte :

| Contenu observé | Interprétation |
|---|---|
| `scp` présent | Token délégué |
| `roles` avec permissions API, pas de `scp`, `idtyp=app` | Token app-only |
| `roles` métier + `scp` | Token utilisateur avec app roles |

`idtyp=app` est le marqueur fiable d'un token app-only.

## Consentement

Configurer une permission dans API permissions ne l'accorde pas. Le grant de consentement est une opération séparée.

Pour les permissions applicatives Microsoft Graph, le rôle attendu pour le consentement administrateur est plus privilégié que l'administration d'application ordinaire ; **Privileged Role Administrator** ou Global Administrator sont les références importantes à reconnaître dans les scénarios de moindre privilège.

Application Administrator / Cloud Application Administrator gèrent les applications et de nombreux consentements pris en charge, mais ne doivent pas être traités comme suffisants pour tous les app roles Microsoft Graph.

Le **admin consent workflow** permet à un utilisateur bloqué par la politique de consentement de soumettre une demande à des approbateurs.

### Révocation : point important

Révoquer un consentement supprime le grant pour les futures émissions, mais **n'invalide pas rétroactivement un access token déjà émis**.

De même, désactiver l'application ou supprimer un secret empêche les nouvelles authentifications / émissions selon le mécanisme concerné, mais ne garantit pas qu'un JWT déjà remis cessera immédiatement d'être accepté par la ressource.

Il faut donc distinguer :

```text
configuration / grant futur
≠
validité d'un token déjà émis
```

## Client credentials flow

Flux pour démons, back-ends et tâches sans utilisateur :

```http
POST https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

client_id={client_id}
&scope=https://graph.microsoft.com/.default
&client_secret={secret}
&grant_type=client_credentials
```

Pour Microsoft Graph avec l'endpoint v2.0, le scope à reconnaître est :

```text
https://graph.microsoft.com/.default
```

`.default` signifie que le token reflète les permissions applicatives déjà configurées et consenties pour cette ressource.

La preuve peut être un client secret, un certificat ou une federated credential. Préférer certificat / federation lorsque le scénario permet d'éviter un secret symétrique.

## Managed identities

Une managed identity est une identité dont Azure gère les credentials. Elle est représentée dans Entra par un service principal et évite au workload de stocker un secret.

| | System-assigned | User-assigned |
|---|---|---|
| Création | Activée sur une ressource | Ressource Azure autonome |
| Cycle de vie | Lié à la ressource | Indépendant |
| Partage | Une ressource | Plusieurs ressources |
| Réflexe examen | doit mourir avec la ressource | même identité pour plusieurs ressources |

Une managed identity supprime le credential, **pas le besoin d'autorisation**. Elle a encore besoin des rôles Azure RBAC ou permissions applicatives nécessaires sur la ressource cible.

## Managed service accounts

Le study guide cite aussi les **managed service accounts**.

Un MSA/gMSA appartient au monde Windows / AD DS : il sert à exécuter des services avec un compte dont le mot de passe est géré automatiquement par Active Directory.

Un **gMSA** peut être utilisé par plusieurs hôtes autorisés.

Grille rapide :

- workload Azure -> managed identity ;
- service Windows dépendant d'AD DS -> MSA/gMSA ;
- application Entra / SaaS -> service principal ;
- compte utilisateur pour un service -> à éviter sauf contrainte explicite.

## Workload identity federation

Pour un workload hors d'Azure disposant d'un IdP OIDC, la workload identity federation permet d'échanger un jeton externe contre un token Entra sans stocker de secret durable.

Scénarios : GitHub Actions, GitLab CI, Kubernetes, autre cloud, environnement sur site compatible.

La federated credential lie notamment : issuer, subject identifier et audience.

## Assignment required

Dans Enterprise applications > Properties, **Assignment required = Yes** signifie que les principals doivent être affectés pour accéder à l'application selon le scénario.

Ce réglage :

- contrôle l'accès / l'affectation ;
- n'accorde aucun consentement API ;
- ne transforme pas un utilisateur en Owner ;
- ne remplace pas les app roles.

## App roles

Un app role est un rôle métier exposé par l'application, par exemple `Sales.Manager`.

Il peut être assigné aux types de principals autorisés par l'application et apparaît dans le claim `roles` du token concerné.

Ne pas confondre :

- assignment required -> cette identité peut-elle accéder à l'application ?
- app role -> avec quel rôle métier ?
- API permission -> que peut faire l'application sur une API ?

## Enterprise applications : SAML et SCIM

### SAML SSO

Pour une application SaaS SAML, les éléments à reconnaître sont :

- Identifier / Entity ID ;
- Reply URL / ACS ;
- Sign-on URL éventuelle ;
- Name ID et claims ;
- certificat de signature.

Les claims SAML se configurent dans la partie **Single sign-on** de l'Enterprise Application, pas dans API permissions.

### Provisioning SCIM

Le provisioning SCIM automatise les opérations Create / Update / Disable / Delete vers une application cible.

Chaîne mentale :

```text
affectation / scope
→ mapping d'attributs
→ moteur de provisioning
→ application SaaS
```

Pour diagnostiquer :

- **qui a changé le mapping ?** -> Audit logs ;
- **pourquoi Bob n'a-t-il pas été provisionné ?** -> Provisioning logs.

### Application collections

Les application collections regroupent des applications pour améliorer leur présentation / organisation dans les expériences utilisateur. Elles ne remplacent ni l'affectation ni le consentement.

## Application Proxy

Application Proxy publie une application web interne via un **private network connector** qui établit des connexions sortantes.

La préauthentification Microsoft Entra ID permet d'appliquer Conditional Access avant le relais vers l'application interne.

À distinguer de Microsoft Entra Private Access : Application Proxy vise surtout les applications web publiées vers le navigateur ; Private Access couvre plus largement les applications privées et protocoles réseau pris en charge par Global Secure Access.

## Defender for Cloud Apps

Le study guide actuel demande plus que access/session policies.

### Cloud Discovery

Analyse l'usage des applications cloud et aide à identifier le Shadow IT.

### Connected apps

Connecte Defender for Cloud Apps à des services SaaS pris en charge afin d'obtenir télémétrie et capacités de gouvernance.

### Application-enforced restrictions

Certaines applications Microsoft peuvent appliquer leurs propres restrictions selon le contexte fourni par Conditional Access, notamment sur appareils non gérés.

### Conditional Access App Control

Route une session vers Defender for Cloud Apps pour contrôle en temps réel.

- access policy -> autoriser / bloquer l'entrée ;
- session policy -> contrôler téléchargement, copie, impression, etc.

### OAuth app policies

Servent à détecter et gouverner les applications OAuth connectées et leurs permissions / comportements.

### Cloud App Catalog

Catalogue des applications cloud avec informations et scores de risque, utilisé notamment avec Cloud Discovery.

## Gouverner les identités de workload

Les contrôles avancés des service principals incluent selon licence :

- Conditional Access for workload identities ;
- risk detections pour workload identities ;
- gouvernance / access reviews adaptées aux identités de workload.

On ne peut pas imposer une MFA interactive à un daemon comme à un humain.

## Ce qui se joue sur des détails

- Application object et service principal : même Client ID, Object ID différents.
- `scp` = permissions déléguées ; `roles` peut contenir permissions applicatives ou app roles métier selon le token.
- `idtyp=app` = marqueur fiable du token app-only.
- `.default` = réflexe client credentials Graph.
- Révoquer un consentement ne détruit pas rétroactivement les tokens déjà émis.
- Managed identity supprime le secret, pas l'autorisation.
- System-assigned = cycle de vie de la ressource ; user-assigned = identité autonome / partageable.
- MSA/gMSA = service Windows / AD DS, pas managed identity Azure.
- SAML claims = Enterprise Application > Single sign-on.
- SCIM : changement de config -> Audit ; exécution -> Provisioning logs.
- Assignment required contrôle l'accès, pas le consentement.
- Shadow IT -> Cloud Discovery ; risky OAuth app -> OAuth app policy ; score d'app SaaS -> Cloud App Catalog.
