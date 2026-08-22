# Parallèle avec un IAM open source

Cette fiche s'adresse aux personnes qui arrivent sur Microsoft Entra avec une expérience Keycloak, OpenID Connect et OAuth 2.0. Les primitives du protocole sont les mêmes ; ce qui change est le découpage des objets et la couche de gouvernance que Microsoft ajoute par-dessus.

Les rapprochements ci-dessous sont utiles pour se repérer, mais plusieurs sont des approximations. Les écarts sont signalés, parce que ce sont eux qui font rater les questions.

## Ce qui se transpose directement

| Keycloak | Microsoft Entra | Écart |
|---|---|---|
| Realm | Tenant | Un serveur Keycloak héberge plusieurs realms ; un tenant Entra est une instance à part entière |
| Issuer du realm | `https://login.microsoftonline.com/{tenantId}/v2.0` | Aucun |
| JWKS endpoint | JWKS Entra, avec rotation automatique des clés | Aucun, la validation de signature suit le même mécanisme |
| Authorization Code plus PKCE | Flux délégué pour les applications interactives | Aucun, PKCE est également recommandé |
| Client Credentials | Flux app-only pour les identités de workload | Entra impose le scope `.default` |
| Client secret | Client secret d'une app registration | Entra plafonne la durée à 24 mois depuis le portail |
| Audience du token | Ressource ou API cible | Aucun |
| Erreur issuer ou audience invalide | Mêmes erreurs de validation OIDC | Aucun |
| Redirect URI | Redirect URI, sous Authentication | Entra applique une correspondance exacte, sans caractère générique |

## Ce qui se transpose mal

### Un client Keycloak n'est pas une app registration

C'est le rapprochement le plus tentant et le plus trompeur, parce qu'il écrase précisément la distinction que Microsoft impose.

Un client Keycloak est un objet unique, scopé à un realm, qui porte à la fois la définition (URI de redirection, secret, rôles, scopes exposés) et l'instance qui reçoit les affectations et les consentements.

Entra sépare ces deux rôles en deux objets : l'**application object**, qui porte la définition et n'existe que dans le tenant d'origine, et le **service principal**, qui porte l'instance locale dans chaque tenant où l'application est présente. Pour une application mono-tenant, la ressemblance avec Keycloak tient encore. Pour une application multi-tenant, elle disparaît : un seul application object, autant de service principals que de tenants clients, chacun avec ses propres consentements.

Le bon rapprochement est donc : un client Keycloak équivaut à un application object **et** son service principal, fusionnés parce que Keycloak ne connaît pas le multi-tenant au sens d'Entra.

### Les rôles Keycloak ne sont pas les app roles Entra

Keycloak distingue les realm roles, valables partout dans le realm, et les client roles, propres à un client. Entra distingue les **rôles d'annuaire**, qui administrent le tenant, et les **app roles**, qui sont métier et propres à une application.

Le rapprochement correct est : client role Keycloak vers app role Entra. Un realm role Keycloak n'a pas d'équivalent direct, puisqu'un rôle Entra d'annuaire n'est pas un rôle applicatif et n'apparaît pas dans le claim `roles` d'un token applicatif.

### Les grants en base ne sont pas des entitlements

Ce que Keycloak stocke sous forme de grants et de consents correspond dans Entra aux `oauth2PermissionGrant`, pour les consentements délégués, et aux `appRoleAssignment`, pour les app roles. Ce sont des objets d'**autorisation**.

Les entitlements au sens Microsoft, c'est-à-dire les assignments d'access packages, sont une couche au-dessus : ils portent une demande, une approbation, une justification et une date d'expiration. Confondre les deux fait perdre la frontière autorisation contre gouvernance, qui est le cœur du domaine Identity Governance.

### Un reverse proxy n'est pas un proxy d'application

Un reverse proxy applicatif est un composant réseau qu'on déploie et qu'on exploite. Le proxy d'application Microsoft Entra est un **service cloud** dont le composant local, un private network connector, n'ouvre aucun port entrant et n'établit que des connexions sortantes. Il ne publie que du HTTP et du HTTPS, et applique la préauthentification Entra donc le Conditional Access.

Pour un accès en RDP, SSH ou tout autre protocole TCP ou UDP, l'équivalent n'est pas le proxy d'application mais **Microsoft Entra Private Access**, qui exige un client sur le poste.

### Les scopes Keycloak ne recouvrent pas les permissions Entra

Un scope OAuth Keycloak est un scope. Entra distingue deux objets là où Keycloak n'en a qu'un : les `oauth2PermissionScope`, permissions déléguées exposées dans `scp`, et les `appRole`, permissions applicatives exposées dans `roles`. La conséquence pratique est que le même intitulé de permission, `User.Read.All`, désigne deux objets différents selon le type, ce qui n'a pas d'équivalent Keycloak.

## Ce que Keycloak n'a pas

Ces briques n'ont pas d'équivalent natif et constituent l'essentiel de ce qu'il faut apprendre en venant de l'open source.

**Le Conditional Access.** Keycloak dispose de flux d'authentification conditionnels et de politiques d'étape, mais rien qui agrège à cette échelle les signaux d'appareil, de conformité Intune, de risque calculé et d'emplacement réseau, avec un mode report-only et une simulation.

**Le risque calculé.** Microsoft Entra ID Protection produit des scores de risque utilisateur et de connexion à partir de télémétrie globale, y compris des identifiants retrouvés dans des fuites. C'est un service de données autant qu'un moteur de règles.

**La gouvernance du cycle de vie.** Entitlement Management, les access reviews et les Lifecycle Workflows répondent à des questions qu'un IdP ne pose pas : pourquoi cet accès existe, qui l'a approuvé, quand expire-t-il, qui le recertifie. Dans un environnement Keycloak, ces fonctions sont généralement assurées par un outil IGA tiers ou par des scripts et des exports CSV.

**L'élévation temporaire de privilège.** PIM, avec son éligibilité, son approbation, sa durée limitée et son authentication context à l'activation, n'a pas d'équivalent.

**Les identités de workload managées.** Une managed identity supprime entièrement le credential, ce qui suppose une intégration avec la plateforme d'exécution que Keycloak, indépendant de toute infrastructure, ne peut pas offrir.

## Ce que l'expérience OIDC apporte réellement

Elle évite les erreurs de raisonnement les plus coûteuses sur ce type d'examen.

La lecture d'un token reste la même : vérifier l'issuer, l'audience, l'expiration, et lire les claims. Savoir qu'un token porte ses autorisations au moment de son émission explique directement pourquoi révoquer un consentement ne coupe pas les sessions en cours, et pourquoi la continuous access evaluation existe.

La distinction entre authentification et autorisation, acquise sur un IdP, se transpose sans effort et évite la confusion entre « obtenir un token » et « avoir le droit d'appeler l'API ».

Le débogage 401 contre 403 se transpose aussi : un 401 renvoie à l'authentification, donc aux sign-in logs ; un 403 renvoie à l'autorisation, donc au consentement, aux app roles ou aux affectations.

Ce qui reste à acquérir n'est pas le protocole, c'est la profondeur produit : le nom exact des objets, les rôles requis pour chaque opération, les frontières de licence, et la couche de gouvernance.
