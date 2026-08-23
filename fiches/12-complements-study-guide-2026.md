# Compléments du study guide SC-300 2026

Cette fiche complète les objectifs explicites du study guide SC-300 en vigueur depuis le 27 avril 2026 qui étaient peu développés dans les autres fiches. Elle reste volontairement orientée examen : objectif, mécanisme à connaître, et piège qui tranche.

Référence : [Study guide officiel SC-300](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/sc-300).

## Domaines personnalisés et Company Branding

### Domaines personnalisés

Un tenant possède toujours un domaine initial en `*.onmicrosoft.com`. Pour utiliser un domaine comme `contoso.com`, il faut l'ajouter au tenant puis **prouver qu'on le possède** auprès du DNS public.

La vérification utilise typiquement un enregistrement **TXT** ; Microsoft peut aussi proposer un enregistrement **MX**. Tant que la vérification n'est pas terminée, le domaine ne peut pas être utilisé normalement comme suffixe d'identité vérifié.

À retenir pour l'examen :

- domaine ajouté != domaine vérifié ;
- la preuve de propriété se fait dans le DNS public ;
- **TXT ou MX** sont les réponses classiques pour la vérification ;
- le domaine initial `onmicrosoft.com` ne peut pas être supprimé.

### Company Branding

Company Branding personnalise l'expérience de connexion Microsoft Entra : logo, image d'arrière-plan, texte de la page de connexion et éléments de marque. Le branding peut être localisé par langue.

Ce réglage agit sur **l'expérience de connexion**, pas sur l'autorisation ni sur le Conditional Access. Une question qui demande de personnaliser l'écran de connexion attend Company Branding, pas une application d'entreprise.

## Tenant, attributs et opérations en masse

### Custom security attributes

Les **custom security attributes** sont des attributs métier personnalisés, définis dans des attribute sets puis affectés à des objets pris en charge. Ils servent notamment à porter des métadonnées exploitables dans des scénarios d'autorisation et de gouvernance.

Ils ne sont pas équivalents à une extension de schéma AD DS ni à une simple propriété libre ajoutée arbitrairement à un utilisateur.

### Bulk operations

Microsoft Entra permet des opérations en masse depuis le portail et par PowerShell / Microsoft Graph. Pour les invités B2B, l'import CSV d'invitations utilise notamment les colonnes :

- `inviteeEmail` : adresse de l'invité ;
- `inviteRedirectUrl` : URL vers laquelle l'utilisateur est redirigé après acceptation.

Le point d'examen est souvent de reconnaître les **noms exacts des colonnes** ou de choisir entre opération manuelle et opération en masse.

### Device registration et device join

Trois états à distinguer :

| État | Cas typique | Relation à l'organisation |
|---|---|---|
| Microsoft Entra registered | BYOD / appareil personnel | enregistré, pas joint |
| Microsoft Entra joined | appareil d'entreprise cloud-first | joint directement à Entra |
| Hybrid Microsoft Entra joined | appareil joint à AD DS local | joint à AD DS et enregistré dans Entra |

`Managed` et `Compliant` sont deux signaux différents : un appareil peut être géré mais non conforme.

## Migration depuis AD FS

Le study guide demande de savoir migrer d'AD FS vers d'autres mécanismes d'authentification et d'autorisation.

Le raisonnement attendu n'est pas de conserver la fédération par habitude. Les choix modernes sont généralement :

- **Password Hash Synchronization (PHS)** pour réduire la dépendance à l'infrastructure locale ;
- **Pass-through Authentication (PTA)** si le mot de passe doit être validé en temps réel contre AD DS ;
- Seamless SSO en complément sur les postes AD DS joints lorsque pertinent.

Une migration se prépare en vérifiant les domaines fédérés, les méthodes d'authentification actuelles, les applications dépendantes d'AD FS et la résilience. PHS peut être activé en parallèle avant le basculement afin d'offrir une voie de secours.

## Authentification : détails qui tombent facilement

### Certificate-Based Authentication et X509SKI

Microsoft Entra CBA authentifie directement avec un certificat X.509. Pour un **high-affinity binding**, il faut reconnaître les identifiants qui lient fortement le certificat à l'utilisateur.

Le raccourci d'examen à retenir est :

> CBA + high affinity -> **X509SKI / Subject Key Identifier (SKI)** -> correspondance via `certificateUserIds`.

Une simple correspondance par UPN est moins spécifique qu'une liaison à une clé ou un certificat précis.

### Windows Hello for Business et Cloud Kerberos Trust

Windows Hello for Business fournit une authentification passwordless liée à l'appareil. En environnement hybride, **Cloud Kerberos Trust** permet aux utilisateurs authentifiés avec Windows Hello for Business d'obtenir l'accès Kerberos aux ressources AD DS sans déployer le modèle plus lourd de certificate trust.

À retenir :

- Windows Hello for Business != simple PIN local ;
- le PIN déverrouille une clé protégée sur l'appareil ;
- Cloud Kerberos Trust relie l'authentification cloud à l'accès Kerberos aux ressources locales ;
- l'objet Microsoft Entra Kerberos est utilisé dans ce scénario hybride.

### Désactiver un compte et révoquer les sessions

Désactiver un compte empêche les nouvelles authentifications, mais un token déjà émis peut continuer d'être présenté tant que la ressource l'accepte. Pour révoquer les sessions utilisateur via Microsoft Graph PowerShell, la commande à reconnaître est :

```powershell
Revoke-MgUserSignInSession -UserId <id>
```

Avec **Continuous Access Evaluation**, les ressources et clients compatibles peuvent réagir à certains événements critiques, comme la désactivation d'un compte, et rejeter un access token avant son expiration normale.

### Registration campaigns

Les campagnes d'enregistrement servent à pousser les utilisateurs à enregistrer une méthode moderne, notamment Microsoft Authenticator. Elles complètent la policy de méthodes : autoriser une méthode ne signifie pas que les utilisateurs l'ont effectivement enregistrée.

## Conditional Access : device-enforced restrictions

Le study guide nomme explicitement les **device-enforced restrictions**. Elles permettent de limiter le comportement d'une application en fonction du contexte de l'appareil, par exemple empêcher certains usages sur un appareil non géré.

Ne pas les confondre avec :

- un **grant control** comme `Require device to be marked as compliant`, qui décide si l'accès est accordé ;
- une **session policy Defender for Cloud Apps**, qui agit via le reverse proxy pendant la session.

## Managed service accounts

Le study guide demande de choisir entre managed identities, service principals, user accounts et **managed service accounts**.

Un managed service account appartient au monde Windows / AD DS et sert à exécuter un service avec un compte dont la gestion du mot de passe est automatisée. Un **gMSA** peut être utilisé par plusieurs hôtes autorisés.

Grille rapide :

- workload Azure -> managed identity en priorité ;
- application Entra / SaaS / daemon -> service principal ;
- service Windows dépendant d'AD DS -> managed service account / gMSA ;
- user account pour un service -> dernier recours, car il recrée des secrets et des cycles de vie humains.

## Enterprise applications : SaaS, SAML et provisioning SCIM

### SSO SAML

Pour une application SaaS ne parlant pas OpenID Connect, Microsoft Entra peut fournir un SSO **SAML**.

Les éléments à reconnaître sont :

- **Identifier / Entity ID** : identifiant du service provider ;
- **Reply URL / Assertion Consumer Service (ACS)** : endpoint où Entra envoie l'assertion ;
- **Sign on URL** : URL de connexion éventuelle ;
- claims et Name ID : données utilisateur envoyées dans l'assertion ;
- certificat de signature SAML : permet à l'application de vérifier l'assertion.

Le piège principal est de modifier les claims dans la configuration **Single sign-on** de l'Enterprise Application, pas dans API permissions.

### Provisioning SCIM

Le provisioning automatique vers une application SaaS utilise fréquemment **SCIM**.

Chaîne à retenir :

1. affectation ou scope de provisioning ;
2. mapping d'attributs source -> cible ;
3. moteur de provisioning ;
4. opérations Create / Update / Disable / Delete vers l'application ;
5. diagnostic dans les **Provisioning logs**.

Si on demande **qui a modifié le mapping**, aller dans les **Audit logs**. Si on demande **pourquoi l'utilisateur n'a pas été créé**, aller dans les **Provisioning logs**.

### Application collections

Les **application collections** regroupent des applications pour les présenter et les gérer comme un ensemble dans les expériences utilisateur. Elles améliorent la découvrabilité ; elles ne remplacent ni l'affectation, ni le consentement, ni les app roles.

## Defender for Cloud Apps : couverture complète du study guide

### Cloud Discovery

Cloud Discovery analyse l'utilisation des applications cloud à partir de sources de trafic et les classe selon leur niveau de risque. Le but est d'identifier le Shadow IT, pas de gérer les permissions OAuth d'une application Entra.

### Connected apps

Defender for Cloud Apps se connecte à des services SaaS pris en charge via des connecteurs afin de récupérer de la télémétrie et d'appliquer certaines fonctions de gouvernance.

### Application-enforced restrictions

Pour certaines applications Microsoft, des restrictions peuvent être imposées à partir du contexte Conditional Access afin de limiter l'expérience sur des appareils non gérés. C'est différent du reverse proxy Conditional Access App Control.

### Conditional Access App Control

Une policy Conditional Access peut router la session vers Defender for Cloud Apps avec le session control **Conditional Access App Control**. C'est le prérequis des contrôles en temps réel par reverse proxy.

- **Access policy** : autoriser ou bloquer l'accès ;
- **Session policy** : contrôler les actions pendant la session, comme téléchargement, copie ou impression.

### OAuth app policies

Les policies OAuth servent à détecter et gouverner les applications OAuth connectées : application à risque, permissions excessives, comportement suspect, révocation ou marquage selon la policy.

À ne pas confondre avec une policy Conditional Access, qui décide de l'accès d'une identité à une ressource.

### Cloud App Catalog

Le Cloud App Catalog contient les applications cloud connues et leurs scores / caractéristiques de risque. Il alimente Cloud Discovery et aide à décider quelles applications sanctionner ou bloquer.

## Terms of Use

Les **Terms of Use (ToU)** présentent un document ou des conditions qu'un utilisateur doit accepter avant l'accès. Ils s'intègrent avec Conditional Access.

Scénario typique :

> imposer l'acceptation d'un document PDF avant d'accéder à une application.

Réponse : **Terms of Use + Conditional Access policy**.

Ne pas confondre ToU avec Access Package : le package attribue un accès ; ToU exige une acceptation juridique ou organisationnelle avant l'accès.

## Lifecycle des externes

Entitlement Management peut gérer le cycle de vie des invités liés aux access packages : invitation / création de l'identité au moment de l'accès, expiration de l'assignment et, selon la configuration et l'absence d'autres assignments, nettoyage ultérieur du compte externe.

La phrase à éviter est donc « un access package ne retire jamais d'accès ». Une **access review** recertifie un accès existant ; un **access package** peut attribuer puis retirer les ressources de son assignment à l'expiration ou au retrait de cette assignment.

## Access Reviews : opérations manuelles

Outre la création de la revue, il faut savoir :

- surveiller son avancement et les décisions ;
- répondre manuellement lorsqu'on est reviewer ;
- appliquer les résultats manuellement si `Auto apply results to resource` est désactivé ;
- distinguer `Deny`, `Approve`, `Don't know` et l'absence de réponse selon la configuration.

## PIM pour Azure resources et audit

PIM ne couvre pas uniquement les rôles Microsoft Entra. Il couvre aussi les **Azure resource roles** : management group, subscription, resource group et ressource.

Le même modèle s'applique :

- affectation active ou éligible ;
- durée ;
- MFA / justification / approbation selon les settings ;
- activation temporaire ;
- historique et audit PIM pour analyser les activations et affectations.

Une question demandant « qui a activé un rôle privilégié pendant les 20 derniers jours » oriente vers l'**audit / history PIM**, pas vers les sign-in logs.

## Micro-fiche de dernière minute

| Formulation | Réflexe |
|---|---|
| verify custom domain | TXT ou MX |
| bulk B2B CSV | `inviteeEmail`, `inviteRedirectUrl` |
| CBA high-affinity binding | X509SKI / SKI |
| Windows Hello hybrid access to Kerberos resources | Cloud Kerberos Trust |
| revoke user sessions | `Revoke-MgUserSignInSession` |
| daemon Graph | client credentials + `https://graph.microsoft.com/.default` + `roles` |
| SAML claim mapping | Enterprise Application > Single sign-on |
| provisioning failure | Provisioning logs |
| who changed provisioning config | Audit logs |
| PDF acceptance before app access | Terms of Use + Conditional Access |
| risky OAuth apps | Defender for Cloud Apps OAuth app policies |
| shadow IT | Cloud Discovery |
| catalogue / risk score of SaaS apps | Cloud App Catalog |
