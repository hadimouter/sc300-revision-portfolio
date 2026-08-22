# Identités de workload

Domaine « Plan and implement workload identities », 20 à 25 % de l'examen.

Une identité de workload est une identité qui n'appartient pas à un humain : une application, un service, un script, une machine. Microsoft Entra en distingue trois formes, et l'essentiel des questions de ce domaine consiste à savoir laquelle choisir et comment lui accorder le strict nécessaire.

## Application object et service principal

Enregistrer une application crée **deux objets distincts**, et c'est la confusion la plus fréquente du domaine.

L'**application object** est la définition globale : identifiants, URI de redirection, secrets et certificats, permissions demandées, API exposée, app roles. Il n'existe qu'une fois, dans le tenant qui a créé l'application. Il est visible sous App registrations.

Le **service principal** est l'instance locale de cette application dans un tenant donné. Il porte les affectations d'utilisateurs, les consentements accordés, les app role assignments et les propriétés de connexion propres à ce tenant. Il est visible sous Enterprise applications.

Pour une application mono-tenant, les deux objets vivent dans le même tenant. Pour une application multi-tenant, il existe un seul application object dans le tenant éditeur, et un service principal par tenant client, créé au moment du premier consentement.

Les deux objets partagent le même **Application (client) ID**, mais ont des **Object ID** différents. C'est le test le plus simple pour vérifier qu'on regarde bien deux objets et non deux vues du même.

| | App registrations | Enterprise applications |
|---|---|---|
| Objet manipulé | Application object | Service principal |
| Ce qu'on y configure | Branding and properties, Authentication (plateformes, URI de redirection, front-channel logout), Certificates and secrets, Token configuration, API permissions, Expose an API, App roles, Owners, Manifest | Properties (Assignment required, Visible to users), Users and groups, Single sign-on, Provisioning, Permissions accordées, Sign-in logs du principal |
| Portée | Le tenant d'origine | Chaque tenant où l'application est présente |

Les URI de redirection se configurent sous **Authentication**, pas ailleurs. Un service principal peut aussi exister sans application object dans le tenant : c'est le cas des applications Microsoft préintégrées et des managed identities.

## Permissions déléguées et permissions applicatives

| | Permission déléguée | Permission applicative |
|---|---|---|
| Contexte | L'application agit au nom d'un utilisateur connecté | L'application agit avec sa propre identité, sans utilisateur |
| Objet Entra | `oauth2PermissionScope` | `appRole` |
| Claim du token | `scp` | `roles` |
| Qui peut consentir | L'utilisateur pour lui-même, sauf permissions admin-restricted ; ou un administrateur pour tous | Un administrateur uniquement, sans exception |
| Portée effective | Intersection des permissions de l'application et des droits de l'utilisateur | Les permissions de l'application, sur tout le tenant |

Le dernier point est déterminant. Une permission déléguée `User.ReadWrite.All` portée par un utilisateur sans droit d'écriture sur l'annuaire ne permet rien : les droits effectifs sont l'intersection. La même permission en applicatif s'applique à l'ensemble du tenant, sans garde-fou. C'est pourquoi les app roles Microsoft Graph sont considérés comme privilégiés.

### Le claim `roles` n'est pas réservé aux tokens app-only

Le raccourci « `scp` égale délégué, `roles` égale applicatif » est vrai dans un sens et faux dans l'autre. Un token app-only ne contient jamais `scp` et porte ses permissions dans `roles`. Mais un token **utilisateur** peut lui aussi contenir `roles` : il y porte alors les **app roles assignés à cet utilisateur**, ou à un de ses groupes, sur l'application cible. Ce sont des rôles applicatifs métier, du type `Sales.Manager`, pas des permissions API.

La lecture correcte d'un token est donc :

| Contenu observé | Interprétation |
|---|---|
| `scp` présent, `sub` correspondant à un utilisateur | Token délégué, permissions API déléguées |
| `roles` présent avec des noms de permissions API, pas de `scp`, `idtyp` valant `app` | Token app-only |
| `roles` présent avec des noms de rôles métier, `scp` également présent | Token délégué portant les app roles de l'utilisateur |

Le claim `idtyp` valant `app` est le marqueur fiable d'un token app-only.

### Consentement

Configurer une permission dans API permissions ne l'accorde pas. Tant que le consentement n'est pas donné, la colonne Status affiche un avertissement et l'appel échoue.

Le rôle nécessaire pour accorder le consentement administrateur dépend de l'API :

| Permission à consentir | Rôle suffisant |
|---|---|
| N'importe quelle permission de n'importe quelle API, sauf les app roles Microsoft Graph | Application Administrator ou Cloud Application Administrator |
| Les app roles Microsoft Graph | Privileged Role Administrator ou Global Administrator |

C'est une exception à retenir : les deux rôles d'administration d'applications sont impuissants sur les permissions applicatives Microsoft Graph, précisément parce qu'elles donnent un accès non filtré à l'annuaire.

Un consentement administrateur accordé au niveau tenant sur des permissions **déléguées** les accorde à tous les utilisateurs, ce qui se matérialise par un `oauth2PermissionGrant` avec `consentType` valant `AllPrincipals` : plus aucun écran de consentement à la connexion. À l'inverse, certaines permissions déléguées sont **admin-restricted** et ne peuvent jamais être consenties par l'utilisateur lui-même, notamment `User.Read.All`, `Group.Read.All` et `Directory.ReadWrite.All`.

Le comportement par défaut du tenant se règle dans Enterprise applications puis Consent and permissions. On peut interdire tout consentement utilisateur, l'autoriser pour les permissions à faible impact seulement, ou l'autoriser largement. Le **admin consent workflow** permet à un utilisateur bloqué de soumettre une demande à des approbateurs désignés plutôt que d'être simplement refusé.

Révoquer un consentement supprime le grant, mais **n'invalide pas les access tokens déjà émis** : ils restent valides jusqu'à expiration, environ une heure. Pour couper immédiatement, il faut désactiver le service principal ou révoquer les sessions.

## Client credentials flow

C'est le flux d'authentification des workloads sans utilisateur : démons, tâches planifiées, back-ends.

L'application présente son `client_id` et une preuve de possession au token endpoint de son tenant, et reçoit un access token portant ses app roles dans le claim `roles`.

```
POST https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

client_id={client_id}
&scope=https://graph.microsoft.com/.default
&client_secret={secret}
&grant_type=client_credentials
```

Le scope `.default` est **obligatoire** dans ce flux, ce n'est pas une commodité. Demander des permissions individuelles, par exemple `scope=https://graph.microsoft.com/User.Read.All`, provoque une erreur. `.default` signifie « toutes les permissions applicatives déjà configurées et consenties pour cette ressource », ce qui implique que le contenu du token dépend entièrement de ce qui a été consenti en amont, et non de ce que le code demande.

La preuve de possession peut être un **client secret**, un **certificat**, ou une **federated credential**. Le secret est le plus simple et le pire : il expire (24 mois au maximum depuis le portail), il se copie, et sa valeur n'est affichable qu'une fois. Le certificat est préférable. La federated credential supprime le problème.

## Managed identities

Une managed identity est une identité de workload dont Azure gère intégralement les identifiants. Elle est représentée dans Entra par un service principal, mais **sans application object et sans credential manipulable**. Il n'y a rien à créer, rien à stocker, rien à faire tourner : c'est la raison d'être de la fonctionnalité, et la réponse attendue chaque fois qu'un scénario mentionne un workload Azure qui doit accéder à une ressource sans secret en configuration.

| | System-assigned | User-assigned |
|---|---|---|
| Création | Activée sur une ressource Azure | Ressource Azure autonome |
| Cycle de vie | Lié à la ressource, supprimée avec elle | Indépendant |
| Partage | Une seule ressource | Plusieurs ressources |
| Cas d'usage | Ressource unique, identité jetable | Flotte de ressources devant partager les mêmes accès, ou pré-attribution des droits avant création de la ressource |

Une managed identity qui appelle Microsoft Graph a toujours besoin d'app roles Graph consentis sur son service principal. Elle supprime le problème du credential, pas celui de l'autorisation, et l'attribution passe par Microsoft Graph ou PowerShell puisque le portail n'expose pas cet écran pour les managed identities.

## Workload identity federation

C'est la réponse pour un workload qui tourne **hors d'Azure** et doit accéder à des ressources Entra sans secret : GitHub Actions, GitLab CI, un cluster Kubernetes non AKS, un autre cloud, ou une charge sur site.

Le principe est un échange de jetons. Le workload obtient un token de son propre fournisseur d'identité, par exemple le token OIDC que GitHub émet pour un job. Il le présente à Entra, qui vérifie que l'émetteur et le sujet correspondent à une **federated credential** déclarée sur l'application, et rend en échange un access token Entra. Aucun secret n'est jamais stocké côté workload.

La configuration se fait sous App registrations puis Certificates and secrets puis Federated credentials, en déclarant l'issuer, le subject identifier et l'audience.

La grille de décision complète tient en trois lignes : sur Azure, une managed identity ; hors d'Azure avec un IdP émettant des jetons OIDC, une workload identity federation ; en dernier recours seulement, un certificat, puis un secret.

## Contrôler l'accès à une application

### Assignment required

Dans Enterprise applications puis Properties, le réglage **Assignment required** détermine qui peut obtenir un token pour l'application. À Yes, seuls les utilisateurs, groupes et service principals explicitement affectés dans Users and groups y parviennent ; les autres reçoivent une erreur d'affectation, même s'ils sont authentifiés et même si le consentement est accordé.

Trois précisions :

Le réglage porte sur « qui ou quoi » : d'autres applications et services sont également soumis à l'affectation, pas seulement des utilisateurs.

Il n'accorde aucun consentement. Affecter un utilisateur à une application ne consent à aucune permission API.

Il a en revanche un effet sur le consentement dans un sens : dès qu'une application exige l'affectation, le consentement utilisateur en libre-service ne suffit plus, puisque l'utilisateur reste bloqué par l'absence d'affectation.

L'affectation **par groupe** exige Microsoft Entra ID P1 ou P2. En édition Free, seules les affectations individuelles fonctionnent. C'est un discriminant récurrent.

### App roles

Un app role est un rôle applicatif défini dans le manifeste de l'application, sous App roles. Il porte un `value` (par exemple `Sales.Manager`), un `displayName` et un `allowedMemberTypes` qui vaut `User`, `Application`, ou les deux.

L'affectation crée un `appRoleAssignment` et se traduit dans le token par une entrée du claim `roles`. C'est le mécanisme par lequel une application externalise son autorisation vers Entra plutôt que de gérer sa propre table de rôles.

Il ne faut pas confondre l'affectation d'accès, qui répond à « cette identité peut-elle obtenir un token pour cette application », et l'app role, qui répond à « avec quel rôle métier ». Une identité peut être affectée sans app role si l'application n'en définit aucun.

## Proxy d'application

Le proxy d'application publie une application web interne, en HTTP ou HTTPS, vers des utilisateurs distants, sans ouvrir de port entrant. Un **Microsoft Entra private network connector** installé sur le réseau interne n'établit que des connexions sortantes vers le service, qui relaie les requêtes.

Deux points testés :

La fonctionnalité exige **Microsoft Entra ID P1 ou P2**. C'est le prérequis le plus souvent posé en question.

La préauthentification a deux valeurs. **Microsoft Entra ID** fait authentifier l'utilisateur par Entra avant tout relais vers l'application interne, ce qui permet d'appliquer le Conditional Access. **Passthrough** relaie directement, sans authentification préalable ni Conditional Access. Un scénario qui veut appliquer la MFA sur une application interne exige donc la préauthentification Entra.

Le connecteur est le même que celui de Microsoft Entra Private Access. Pour de nouveaux déploiements, Private Access est la direction recommandée par Microsoft, le proxy d'application restant pertinent pour publier une URL web accessible depuis un navigateur non équipé de client.

## Gouverner les identités de workload

Les identités de workload disposent de leurs propres contrôles, regroupés dans la licence **Microsoft Entra Workload ID Premium**, distincte de P1 et P2.

Le **Conditional Access for workload identities** applique des policies aux service principals, sur des signaux réduits : emplacement réseau et niveau de risque. On ne peut pas exiger de MFA d'un démon.

La **détection de risque sur les identités de workload** signale des identifiants divulgués, des connexions depuis des adresses suspectes ou des schémas d'authentification anormaux.

Les **access reviews d'identités de workload** permettent de recertifier les service principals affectés à des rôles privilégiés ou à des applications.

En complément, Microsoft Graph permet d'auditer les credentials arrivant à expiration :

```
GET /applications?$select=id,displayName,passwordCredentials,keyCredentials
```

## Ce qui se joue sur des détails

Un application object et un service principal partagent le Client ID mais pas l'Object ID.

Une permission applicative exige toujours un consentement administrateur, sans exception, et les app roles Microsoft Graph exigent Privileged Role Administrator ou Global Administrator.

Le nom d'une permission ne suffit pas : `User.Read.All` existe en délégué et en applicatif, et les deux ne donnent pas du tout la même chose.

`roles` dans un token utilisateur porte des app roles métier, pas des permissions applicatives. Seul `idtyp` valant `app` identifie de façon fiable un token app-only.

Le scope `.default` est obligatoire dans le client credentials flow, et le contenu du token dépend du consentement, pas de la demande.

Révoquer un consentement ne coupe pas les tokens en cours de validité.

Assignment required contrôle l'accès, pas le consentement, et l'affectation par groupe exige P1.

Le proxy d'application exige P1, et seule la préauthentification Entra permet d'appliquer le Conditional Access.
