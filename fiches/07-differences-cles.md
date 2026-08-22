# Différences clés

Chaque entrée oppose deux notions que l'examen confond volontiers, en donnant le critère qui tranche et la façon dont la question est habituellement posée.

## App registration et enterprise application

Ce sont deux vues sur deux objets différents, pas deux vues sur le même objet.

App registrations manipule l'application object, c'est-à-dire la définition : URI de redirection sous Authentication, secrets et certificats, permissions demandées, API exposée, app roles. Enterprise applications manipule le service principal, c'est-à-dire l'instance locale : affectations d'utilisateurs et de groupes, single sign-on, provisioning, permissions consenties, connexions du principal.

Ce qui tranche : même Client ID, Object ID différents. Une application multi-tenant a un application object et autant de service principals que de tenants clients.

Formulation typique : « où configurez-vous les URI de redirection » ou « où affectez-vous des utilisateurs à l'application ».

## Permission déléguée et permission applicative

La délégation fait agir l'application au nom d'un utilisateur ; l'applicative la fait agir seule.

Ce qui tranche vraiment n'est pas le claim mais la portée effective. En délégué, les droits sont l'**intersection** des permissions de l'application et de ceux de l'utilisateur : une permission `User.ReadWrite.All` déléguée portée par un utilisateur sans droit d'écriture ne donne rien. En applicatif, la permission s'applique à tout le tenant, sans garde-fou. C'est la raison pour laquelle les app roles Microsoft Graph exigent Privileged Role Administrator et non Application Administrator.

Formulation typique : « without a signed-in user » impose l'applicative, « on behalf of the signed-in user » impose la déléguée.

## Claim `scp` et claim `roles`

`scp` n'apparaît que dans les tokens délégués. `roles` apparaît dans les tokens app-only, où il porte les permissions applicatives, **et** dans les tokens utilisateur, où il porte les app roles métier assignés à l'utilisateur ou à ses groupes.

Ce qui tranche : le claim `idtyp` valant `app` identifie un token app-only. La seule présence de `roles` ne prouve rien.

## Assignment required et admin consent

Assignment required contrôle **qui peut obtenir un token** pour l'application. Le consentement contrôle **ce que l'application a le droit de faire** sur une API.

Les deux sont indépendants dans un sens : affecter un utilisateur ne consent à rien. Dans l'autre sens il existe un lien, souvent oublié : dès qu'une application exige l'affectation, le consentement utilisateur en libre-service ne suffit plus à débloquer l'accès, puisque l'absence d'affectation bloque en amont.

Ce qui tranche : l'affectation par groupe exige P1, l'affectation individuelle non.

## Managed identity et workload identity federation

Les deux suppriment le secret. La localisation du workload décide.

Une managed identity ne fonctionne que pour un workload qui tourne **sur Azure**, puisque c'est la plateforme Azure qui fournit le jeton. Une workload identity federation couvre tout workload disposant d'un IdP émettant des jetons OIDC : GitHub Actions, GitLab, Kubernetes hors AKS, un autre cloud, un serveur sur site.

Formulation typique : un pipeline GitHub Actions qui doit déployer sur Azure sans secret attend la federation, pas la managed identity.

## Managed identity system-assigned et user-assigned

La system-assigned naît et meurt avec la ressource, et ne sert qu'à elle. La user-assigned est une ressource Azure autonome, partageable entre plusieurs workloads, et survit à leur suppression.

Ce qui tranche : « la même identité pour plusieurs ressources » ou « attribuer les droits avant de créer la ressource » imposent la user-assigned. « L'identité doit disparaître avec la ressource » impose la system-assigned.

## Application Proxy et Microsoft Entra Private Access

Les deux publient des ressources internes via le même private network connector, sans ouvrir de port entrant.

Le proxy d'application publie une **URL web** accessible depuis n'importe quel navigateur, en HTTP et HTTPS uniquement. Private Access donne accès à des applications privées **tous protocoles TCP et UDP**, sans publier d'URL, et exige un client Global Secure Access sur le poste.

Ce qui tranche : le protocole et la présence d'un client. Un besoin de RDP ou SSH exclut le proxy d'application. Un besoin d'accès depuis un navigateur non équipé exclut Private Access.

## Private Access et VPN

Un VPN place le poste sur le réseau et lui donne, par défaut, la visibilité de ce réseau. Private Access accorde l'accès **application par application**, chaque application publiée pouvant porter ses propres policies Conditional Access, et ne donne aucune visibilité sur le reste.

Ce qui tranche : la granularité de l'accès et la capacité d'appliquer du Conditional Access par application.

## Conditional Access et Security Defaults

Les Security Defaults sont gratuits, imposent la MFA à tous et ne se règlent pas. Le Conditional Access exige P1 et permet le ciblage fin.

Ce qui tranche n'est pas une préférence mais une contrainte technique : les deux sont **mutuellement exclusifs**. Tant que les Security Defaults sont actifs, la création d'une policy CA est refusée. Un scénario de migration commence donc par leur désactivation.

## Grant control et session control

Un grant control est une condition d'octroi : il décide si l'accès est accordé, et sous quelle exigence. Un session control agit **après** l'octroi, sur la durée et le comportement de la session.

Ce qui tranche : la MFA, l'appareil conforme et l'authentication strength sont des grant controls. Sign-in frequency, persistent browser, continuous access evaluation et Conditional Access App Control sont des session controls.

Formulation typique : « during the session » impose un session control.

## Authentication methods policy et authentication strength

La policy de méthodes définit ce qui est **disponible** dans le tenant. L'authentication strength définit ce qui est **acceptable** pour un accès donné.

Ce qui tranche : une strength exigeant une méthode non activée dans la policy rend l'accès impossible. Et dans une même policy CA, « Require authentication strength » et « Require multifactor authentication » ne peuvent pas être cochés ensemble.

## User risk et sign-in risk

Le risque utilisateur porte sur le **compte** : il agrège des signaux persistants, typiquement des identifiants retrouvés dans une fuite. Le risque de connexion porte sur une **tentative précise** : voyage impossible, adresse IP anonyme, propriétés inhabituelles.

Ce qui tranche : la remédiation attendue. Un risque utilisateur élevé appelle un changement de mot de passe sécurisé ; un risque de connexion élevé appelle une MFA. Les deux exigent P2.

## Access policy et session policy dans Defender for Cloud Apps

L'access policy décide d'entrer ou non. La session policy contrôle ce qui se passe une fois entré : téléchargement, copie, impression, étiquetage des fichiers.

Ce qui tranche, et qui est le vrai piège : **les deux** exigent que l'application soit routée vers le reverse proxy par une policy Conditional Access dont le session control est Conditional Access App Control. Conditional Access App Control n'est pas réservé aux session policies.

## Access package et access review

Un access package sert à **obtenir** un ensemble d'accès, par une demande et une approbation. Une access review sert à **conserver ou retirer** un accès existant, par une recertification.

Ce qui tranche : la direction. Aucun access package ne retire d'accès, aucune access review n'en attribue.

Formulation typique : « demander », « catalogue », « approbation » pour le package ; « périodiquement », « confirmer », « toujours nécessaire » pour la revue.

## Access review et lifecycle workflow

Une access review pose une question à un humain, périodiquement. Un lifecycle workflow exécute des actions automatiquement, à une étape du cycle de vie.

Ce qui tranche : la présence ou non d'une décision humaine, et le déclencheur. Une date d'arrivée ou de départ déclenche un workflow ; un calendrier de recertification déclenche une revue.

Côté licence, les revues exigent P2 et les workflows exigent Microsoft Entra ID Governance.

## Éligible et actif dans PIM

Une affectation active confère les privilèges maintenant. Une affectation éligible ne confère rien tant qu'elle n'est pas activée.

Ce qui tranche : « permanent active » est le privilège permanent qu'on cherche à éliminer ; « éligible » est l'activation à la demande. Les deux peuvent par ailleurs être permanentes ou limitées dans le temps.

## PIM for roles et PIM for Groups

PIM gouverne directement les rôles Entra et les rôles Azure. Pour tout le reste, il faut passer par PIM for Groups : on rend l'utilisateur éligible à devenir membre du groupe, et le groupe porte l'accès.

Ce qui tranche : la nature de l'accès à rendre temporaire. Un accès applicatif, une licence ou un rôle dans une application SaaS ne sont pas des rôles PIM et exigent donc PIM for Groups.

## Security group et administrative unit

Un groupe répond à « qui reçoit un accès ». Une administrative unit répond à « quels objets un administrateur peut gérer ».

Ce qui tranche : le groupe est un vecteur d'autorisation, l'AU est un périmètre d'administration. Et l'AU n'est pas transitive : un groupe placé dans une AU n'y place pas ses membres.

## Owner et member d'un groupe

Un Owner gère le groupe. Un Member bénéficie de ce que le groupe distribue : licences, rôles Azure RBAC, accès applicatifs.

Ce qui tranche : un Owner qui n'est pas Member ne reçoit rien. Mais dans un raisonnement de moindre privilège, il faut voir qu'il peut s'auto-ajouter comme membre, donc qu'il détient un chemin d'escalade vers tout ce que le groupe donne.

## Rôle Entra et rôle Azure RBAC

Les rôles Entra administrent l'annuaire, les rôles Azure RBAC administrent les ressources Azure. Ce sont deux systèmes séparés, avec des portées différentes.

Ce qui tranche : un Global Administrator n'est pas Owner des abonnements. Mais il peut basculer le commutateur « Access management for Azure resources » et recevoir User Access Administrator à la portée racine. Ce n'est ni automatique, ni Owner, et c'est tracé.

## Member et Guest

`UserType` décrit la relation à l'organisation, pas la provenance de l'identité.

Ce qui tranche : accepter une invitation fait passer `externalUserState` à `Accepted`, et ne change pas `UserType`. Et les deux axes se croisent : il existe des internal guests et des external members, ces derniers étant le produit par défaut de la cross-tenant synchronization.

## External collaboration, cross-tenant access et cross-tenant synchronization

Trois réglages souvent cités ensemble, jamais interchangeables.

External collaboration settings fixe les règles générales d'invitation : qui peut inviter, quels domaines, quelles restrictions de lecture d'annuaire.

Cross-tenant access settings définit la relation avec un tenant partenaire identifié, et surtout les trust settings qui permettent d'accepter la MFA ou la conformité d'appareil réalisée chez lui.

Cross-tenant synchronization provisionne automatiquement des comptes d'un tenant vers un autre, sans invitation.

Ce qui tranche : « nos partenaires refont une MFA chez nous » appelle les trust settings des cross-tenant access settings. « Les utilisateurs de notre autre tenant doivent apparaître automatiquement » appelle la synchronisation.

## Connect Sync et Cloud Sync

Le discriminant n'est pas le nombre de forêts : les deux en gèrent plusieurs.

Cloud Sync est le seul à gérer des forêts **déconnectées**, sans relation d'approbation, typiquement après une fusion. Connect Sync est le seul à synchroniser les **appareils**, donc à permettre le Hybrid Entra join, et le seul à couvrir les scénarios Exchange hybrides complets et le filtrage par attribut.

Ce qui tranche : une exigence de hybrid join impose Connect Sync ; deux forêts sans approbation imposent Cloud Sync. Les deux peuvent coexister sur le même annuaire.

## PHS et PTA

PHS synchronise un hash du hash du mot de passe et fait authentifier dans le cloud. PTA valide le mot de passe contre l'Active Directory local via des agents sortants.

Ce qui tranche : la dépendance à l'infrastructure locale. PHS survit à une coupure du site local et alimente la détection de fuites d'identifiants d'ID Protection. PTA ne stocke aucun dérivé de mot de passe dans le cloud mais dépend de la disponibilité des agents.

## Registered, joined et hybrid joined

Le critère est la propriété de l'appareil et la présence d'un Active Directory.

Registered désigne un appareil personnel, apporté par l'utilisateur. Joined désigne un appareil de l'organisation, sans Active Directory local. Hybrid joined désigne un appareil de l'organisation joint à un Active Directory local et enregistré dans Entra, ce qui exige la synchronisation des appareils.

## Managed et compliant

Managed signifie géré par une solution de gestion. Compliant signifie conforme aux règles d'une compliance policy.

Ce qui tranche : un appareil peut être managed sans être compliant, par exemple s'il n'a pas encore appliqué une mise à jour exigée. Un statut « Compliant : No » ne signifie donc ni appareil compromis ni appareil non géré.

## Sign-in, audit et provisioning logs

Trois journaux, trois questions.

Sign-in répond à « qui s'est authentifié ». Audit répond à « qui a modifié la configuration ». Provisioning répond à « qu'a tenté le moteur vers l'application cible ».

Ce qui tranche : « who changed » impose l'audit, « automatic create, update, disable » impose le provisioning, et une question sur une managed identity impose l'onglet dédié des sign-in logs, distinct de celui des service principals.

## Authentification, autorisation et gouvernance

L'authentification établit qui vous êtes. L'autorisation détermine ce que vous pouvez faire. La gouvernance justifie pourquoi cet accès existe, pour combien de temps, et qui le revalide.

Ce qui tranche : dès qu'un énoncé parle d'approbation, d'expiration, de recertification ou de justification, il s'agit de gouvernance, et donc de licences Governance ou P2.

## RBAC et ABAC

Le RBAC accorde des droits selon un rôle attribué à un principal. L'ABAC affine cette décision par des conditions évaluées au moment de l'accès, portant sur des attributs de la ressource, du principal ou de la requête.

Dans Azure, les conditions de role assignment sur le stockage sont une mise en œuvre d'ABAC : le rôle donne l'accès, la condition le restreint aux objets portant une étiquette donnée.
