# Différences clés

Chaque entrée oppose deux notions que l'examen confond volontiers, avec le critère qui tranche.

## App registration et enterprise application

App registrations manipule l'**application object**, c'est-à-dire la définition : URI de redirection, credentials, API permissions, API exposée, app roles.

Enterprise applications manipule le **service principal**, l'instance locale : affectations, SSO, provisioning, consentements et propriétés propres au tenant.

Ce qui tranche : même Client ID, Object ID différents.

## Permission déléguée et permission applicative

La permission déléguée fait agir l'application **au nom d'un utilisateur connecté**. La permission applicative fait agir l'application **avec sa propre identité**, sans utilisateur.

Ce qui tranche : `without a signed-in user` impose l'applicative ; `on behalf of the signed-in user` impose la déléguée.

## Claim `scp` et claim `roles`

`scp` porte les permissions API déléguées.

`roles` peut porter les permissions applicatives d'un token app-only **ou** des app roles métier dans un token utilisateur.

Ce qui tranche : `idtyp=app` identifie fiablement un token app-only. La présence de `roles` seule ne suffit pas.

## Assignment required et admin consent

Assignment required contrôle **qui peut accéder / obtenir un token pour l'application selon les affectations**.

Le consentement contrôle **ce que l'application peut faire sur une API**.

Affecter un utilisateur ne consent à rien. Consentir à une permission ne remplace pas l'affectation si `Assignment required = Yes`.

## Managed identity et workload identity federation

Les deux évitent le secret applicatif classique.

Une managed identity est fournie par Azure pour les ressources prises en charge. La workload identity federation convient à un workload qui présente un jeton d'un IdP externe, par exemple GitHub Actions ou Kubernetes.

Ce qui tranche : workload Azure -> managed identity ; workload hors Azure avec OIDC -> federation.

## Managed identity et managed service account

Une **managed identity** est une identité de workload Azure représentée dans Entra par un service principal, sans credential à gérer par l'application.

Un **managed service account / gMSA** est un compte de service Windows géré par AD DS, avec gestion automatisée du mot de passe.

Ce qui tranche : service Windows dépendant d'AD DS -> MSA/gMSA ; workload Azure -> managed identity.

## Managed identity system-assigned et user-assigned

La system-assigned naît et meurt avec une ressource Azure.

La user-assigned est une ressource autonome, réutilisable par plusieurs workloads et indépendante de leur cycle de vie.

Ce qui tranche : même identité pour plusieurs ressources -> user-assigned ; identité qui doit disparaître avec la ressource -> system-assigned.

## Application Proxy et Microsoft Entra Private Access

Application Proxy publie une application web interne, principalement HTTP/HTTPS, vers un navigateur.

Private Access donne un accès Zero Trust à des applications privées sur des protocoles plus larges, avec le client Global Secure Access et un private network connector.

Ce qui tranche : URL web accessible depuis un navigateur -> Application Proxy ; RDP/SSH/TCP/UDP privé -> Private Access.

## Private Access et VPN

Un VPN donne typiquement une connectivité réseau plus large.

Private Access accorde un accès **application par application** et s'intègre au Conditional Access.

Ce qui tranche : accès au réseau vs accès ciblé à l'application.

## SAML SSO et API permissions OAuth

Le SSO SAML concerne l'authentification vers une application SaaS : Entity ID, Reply URL/ACS, Name ID, claims, certificat de signature.

Les API permissions OAuth concernent l'autorisation d'une application à appeler une API.

Ce qui tranche : modifier le Name ID ou les claims de connexion -> Enterprise Application > Single sign-on ; donner `User.Read.All` -> App Registration > API permissions.

## SCIM provisioning et SSO

Le SSO permet à l'utilisateur de se connecter à l'application.

SCIM automatise la création, mise à jour, désactivation ou suppression des comptes dans l'application cible.

Ce qui tranche : problème de login -> sign-in / SSO ; compte absent de l'application -> provisioning / SCIM.

## Conditional Access et Security Defaults

Security Defaults fournit une protection globale simple et gratuite.

Conditional Access permet un ciblage et des contrôles fins avec les licences correspondantes.

Ce qui tranche : besoin de ciblage par utilisateur, application, risque, appareil ou emplacement -> Conditional Access.

## Grant control et session control

Un grant control décide ce qui doit être satisfait **pour obtenir l'accès** : MFA, authentication strength, appareil conforme, etc.

Un session control agit **après l'octroi**, par exemple sign-in frequency ou Conditional Access App Control.

Ce qui tranche : `during the session` -> session control.

## Authentication methods policy et authentication strength

La policy de méthodes définit ce qui est **autorisé / disponible** pour les utilisateurs.

L'authentication strength définit les combinaisons considérées comme **suffisantes pour un accès donné**.

Ce qui tranche : permettre FIDO2 -> methods policy ; exiger uniquement des méthodes phishing-resistant sur une application -> authentication strength dans CA.

## CBA username binding et high-affinity binding

Un binding simple peut rapprocher un identifiant du certificat d'un attribut utilisateur.

Un high-affinity binding lie plus fortement le certificat à l'identité. À l'examen, **SKI / X509SKI** est un identifiant à reconnaître immédiatement.

## Windows Hello for Business et mot de passe

Windows Hello for Business utilise une clé protégée sur l'appareil ; le PIN sert à déverrouiller cette clé localement.

Ce n'est pas un mot de passe transmis au serveur.

En hybride, Cloud Kerberos Trust permet l'accès Kerberos aux ressources AD DS sans déployer un modèle certificate trust complet.

## User risk et sign-in risk

User risk = probabilité que **le compte** soit compromis.

Sign-in risk = probabilité qu'**une tentative de connexion précise** soit illégitime.

Ce qui tranche : le risque persistant de l'identité vs le risque de l'événement de connexion.

## CAE et expiration normale du token

Sans mécanisme de révocation anticipée applicable, un access token peut rester utilisable jusqu'à expiration.

Avec Continuous Access Evaluation, des clients et ressources compatibles peuvent réagir à certains événements critiques et rejeter un token avant son expiration normale.

Ce qui tranche : compte désactivé + ressource compatible CAE -> ne pas supposer que le token reste forcément accepté jusqu'à `exp`.

## Révocation de consentement et révocation d'un token

Révoquer un consentement retire le grant pour les **futures émissions**.

Cela ne réécrit pas un access token déjà remis. Désactiver l'application ou supprimer un secret ne doit pas non plus être présenté comme une suppression rétroactive de tous les JWT déjà émis.

## Access policy et session policy dans Defender for Cloud Apps

Access policy = autoriser ou bloquer l'entrée.

Session policy = contrôler les actions pendant la session : téléchargement, copie, impression, etc.

Pour les contrôles temps réel par reverse proxy, le trafic doit être routé via Conditional Access App Control.

## Cloud Discovery et OAuth app policies

Cloud Discovery sert à identifier l'usage des applications cloud et le Shadow IT.

Les OAuth app policies servent à gouverner les applications OAuth connectées et leurs permissions / comportements.

Ce qui tranche : `which cloud apps are users using?` -> Cloud Discovery ; `risky OAuth application` -> OAuth app policy.

## Cloud App Catalog et My Apps

Cloud App Catalog = catalogue de services cloud avec informations et scores de risque.

My Apps = portail utilisateur pour accéder aux applications qui lui sont disponibles.

## Access package et access review

Un access package organise **l'attribution et le cycle de vie d'une assignment de ressources** : demande, approbation, durée, expiration.

Une access review sert à **recertifier un accès existant**.

Ce qui tranche : `request/approve/expire` -> access package ; `periodically confirm whether access is still required` -> access review.

Important : l'expiration ou le retrait d'une assignment d'access package **retire bien les accès qu'elle fournissait**. La phrase « un access package ne retire jamais d'accès » est donc fausse.

## Access package et Terms of Use

Access package = obtenir des ressources.

Terms of Use = accepter un document / des conditions avant l'accès, généralement via Conditional Access.

Ce qui tranche : `must accept a PDF before accessing the app` -> Terms of Use, pas Access Package.

## Access review et lifecycle workflow

Une access review pose une question à un humain pour recertifier l'accès.

Lifecycle Workflows exécute automatiquement des tâches à une étape Joiner/Mover/Leaver.

Lifecycle Workflows est une capacité Governance avancée ; ce n'est pas un bullet explicite du study guide SC-300 d'avril 2026, même si le sujet reste pertinent dans l'écosystème IAM.

## P2 et Entra ID Governance

P2 inclut les capacités historiques d'Entitlement Management et d'Access Reviews que Microsoft avait déjà rendues GA dans P2.

Le produit Microsoft Entra ID Governance ajoute des capacités avancées et inclut également ces capacités historiques.

Ce qui tranche : ne plus utiliser le raccourci obsolète « access package = Governance obligatoire dans tous les cas ».

## Éligible et actif dans PIM

Active = privilèges disponibles maintenant.

Eligible = aucun privilège tant que l'utilisateur n'active pas l'affectation.

Les deux peuvent être permanents ou limités dans le temps selon les paramètres.

## PIM pour rôles et PIM for Groups

PIM gouverne les rôles Microsoft Entra et Azure resources.

PIM for Groups rend l'appartenance ou la propriété d'un groupe éligible / temporaire, ce qui permet de rendre temporaires des accès distribués par ce groupe.

## Security group et administrative unit

Un groupe répond à « qui reçoit l'accès ? ».

Une administrative unit répond à « quels objets cet administrateur peut-il gérer ? ».

Et une AU n'est pas transitive : un groupe placé dans une AU n'y place pas automatiquement ses membres utilisateurs.

## Owner et Member d'un groupe

Owner = gère le groupe.

Member = appartient au groupe et reçoit ce que l'appartenance distribue.

Un Owner qui n'est pas Member ne reçoit pas automatiquement l'accès du groupe.

## Rôle Entra et rôle Azure RBAC

Rôle Entra = administration de l'annuaire.

Azure RBAC = autorisation sur les ressources Azure.

Un Global Administrator n'est pas automatiquement Owner d'un abonnement Azure.

## Member et Guest

`UserType` décrit la relation à l'organisation.

Accepter une invitation change l'état d'acceptation mais ne transforme pas automatiquement Guest en Member.

## External collaboration, cross-tenant access et cross-tenant synchronization

External collaboration settings = règles générales B2B et invitation.

Cross-tenant access settings = relation et trust avec un tenant partenaire identifié.

Cross-tenant synchronization = provisioning automatique de comptes entre tenants.

## Connect Sync et Cloud Sync

Ne pas choisir uniquement sur « multi-forêts ».

Cloud Sync excelle dans certains scénarios légers / forêts déconnectées ; Connect Sync reste nécessaire pour des besoins comme la synchronisation des appareils / Hybrid Entra join et des scénarios hybrides plus complets.

## PHS et PTA

PHS synchronise un dérivé du mot de passe vers Entra et permet l'authentification cloud.

PTA valide le mot de passe contre AD DS via les agents au moment de la connexion.

Ce qui tranche : indépendance du site local -> PHS ; validation AD DS temps réel -> PTA.

## Registered, joined et hybrid joined

Registered = généralement BYOD / personnel.

Joined = appareil organisationnel joint directement à Entra.

Hybrid joined = appareil joint à AD DS local et enregistré dans Entra.

## Managed et compliant

Managed = géré par une solution de gestion.

Compliant = satisfait les règles de conformité.

Un appareil peut être managed sans être compliant.

## Sign-in, audit et provisioning logs

Sign-in = authentification.

Audit = changement de configuration.

Provisioning = actions du moteur de provisioning.

Ce qui tranche : `who changed the SCIM mapping` -> Audit ; `why wasn't the user provisioned` -> Provisioning.

## Authentification, autorisation et gouvernance

Authentification = qui es-tu ?

Autorisation = que peux-tu faire ?

Gouvernance = pourquoi cet accès existe-t-il, pour combien de temps et qui le revalide ?

## RBAC et ABAC

RBAC accorde un rôle à un principal sur une portée.

ABAC ajoute des conditions fondées sur des attributs au moment de la décision d'accès.
