# SC-300 — Final Coverage Test

Simulation originale de 50 questions construite pour compléter les sujets peu ou pas couverts par le Practice Assessment Microsoft utilisé pendant la préparation.

Le test ne reproduit aucune question Microsoft. Les scénarios, noms, formulations et distracteurs sont originaux. L'objectif est de vérifier le transfert des connaissances sur les quatre domaines officiels SC-300, avec un niveau de précision proche d'un examen réel.

> Note : ce test suit le périmètre SC-300 en vigueur depuis le 27 avril 2026. Les fonctionnalités en préversion peuvent évoluer plus vite que le contenu de certification.

## Questions

### 1.
Votre organisation utilise des attributs de sécurité personnalisés Microsoft Entra.

Un administrateur doit pouvoir **attribuer des valeurs d'attributs existants à des utilisateurs**, mais ne doit pas pouvoir créer ou modifier les définitions des attributs.

Quel rôle devez-vous lui attribuer ?

A. Attribute Definition Administrator  
B. Attribute Assignment Administrator  
C. User Administrator  
D. Privileged Role Administrator

### 2.
Contoso utilise plusieurs tenants Microsoft Entra et configure la synchronisation automatique des employés du tenant source vers un tenant cible.

Aucun mapping personnalisé de `userType` n'est défini.

Quel type d'utilisateur est créé par défaut dans le tenant cible ?

A. Internal Guest  
B. External Member  
C. External Guest  
D. Internal Member

### 3.
Une entreprise veut afficher son logo, son arrière-plan personnalisé et un texte d'assistance sur l'écran de connexion Microsoft Entra.

Quelle fonctionnalité devez-vous configurer ?

A. Company Branding  
B. Terms of Use  
C. Authentication Context  
D. My Apps collection

### 4.
Un employé utilise son ordinateur Windows personnel.

Il se connecte à Windows avec son compte personnel, puis ajoute son compte professionnel dans **Access work or school** afin d'accéder à Microsoft 365.

Comment l'appareil est-il généralement représenté dans Microsoft Entra ?

A. Microsoft Entra joined  
B. Hybrid Microsoft Entra joined  
C. Microsoft Entra registered  
D. AD DS joined uniquement

### 5.
Une entreprise vient d'acquérir une société dont la forêt Active Directory n'a **aucune relation d'approbation** avec la forêt principale.

Vous voulez synchroniser les utilisateurs des deux forêts vers Microsoft Entra avec une infrastructure locale minimale.

Que devez-vous privilégier ?

A. Microsoft Entra Connect Sync uniquement  
B. Microsoft Entra Cloud Sync  
C. AD FS  
D. Application Proxy

### 6.
Votre organisation veut implémenter **Hybrid Microsoft Entra Join** pour ses ordinateurs Windows joints au domaine AD DS.

Quelle solution de synchronisation devez-vous utiliser pour synchroniser les objets appareils nécessaires dans le scénario classique attendu par le SC-300 ?

A. Microsoft Entra Connect Sync  
B. Microsoft Entra Cloud Sync uniquement  
C. Cross-tenant synchronization  
D. SCIM provisioning

### 7.
Vous attribuez une licence Microsoft 365 à un utilisateur, mais l'attribution échoue alors que des licences sont disponibles.

L'utilisateur vient d'être créé et une propriété géographique obligatoire n'a pas encore été renseignée.

Quelle propriété devez-vous vérifier en priorité ?

A. `department`  
B. `employeeType`  
C. `officeLocation`  
D. `usageLocation`

### 8.
Vous voulez surveiller :

- les problèmes de synchronisation Microsoft Entra Connect ;
- l'état de vos serveurs AD FS ;
- certains contrôleurs de domaine.

Quel service est le plus adapté ?

A. Identity Secure Score  
B. Microsoft Defender for Cloud Apps  
C. Microsoft Entra Connect Health  
D. Azure Policy

### 9.
Une organisation partenaire ne possède pas Microsoft Entra ID mais dispose d'un fournisseur d'identité compatible **SAML 2.0**.

Vous voulez permettre à ses utilisateurs B2B de s'authentifier auprès de ce fournisseur.

Que devez-vous configurer ?

A. Un fournisseur d'identité externe / direct federation  
B. Password Hash Synchronization  
C. Cross-tenant synchronization  
D. Managed Identity

### 10.
Alice possède deux affectations de rôles Microsoft Entra :

- un rôle intégré à l'échelle du tenant ;
- un rôle personnalisé limité à l'unité administrative `AU-France`.

Quelle règle décrit le mieux ses permissions effectives ?

A. Seule l'affectation la plus privilégiée est conservée  
B. Les permissions des affectations se combinent, chacune restant limitée à sa portée  
C. Le rôle personnalisé annule automatiquement le rôle intégré  
D. Une unité administrative transforme les rôles Entra en Azure RBAC

### 11.
Votre organisation utilise Microsoft Entra Certificate-Based Authentication.

Vous voulez mapper un certificat utilisateur grâce à son **Subject Key Identifier**.

Quel identifiant de liaison est pertinent ?

A. `X509SKI`  
B. `oauth2PermissionScope`  
C. `idtyp`  
D. `x5t#S256` uniquement comme app role

### 12.
Un nouvel employé ne possède encore aucune méthode MFA.

Vous voulez lui permettre de s'authentifier temporairement afin qu'il puisse enregistrer ensuite une passkey FIDO2.

Quelle méthode est conçue pour ce scénario ?

A. SMS permanent  
B. Temporary Access Pass  
C. Client Secret  
D. Password Hash Synchronization

### 13.
Une ressource extrêmement sensible doit accepter uniquement des méthodes **MFA résistantes au phishing**.

Quelles méthodes peuvent satisfaire la force d'authentification intégrée correspondante ?  
**Choisissez trois réponses.**

A. Clé de sécurité FIDO2  
B. SMS + mot de passe  
C. Windows Hello for Business  
D. Certificate-Based Authentication configurée comme MFA  
E. Software OATH

### 14.
Votre environnement est hybride.

Les utilisateurs réinitialisent leur mot de passe avec Microsoft Entra SSPR, mais le nouveau mot de passe doit également être écrit dans l'Active Directory local.

Quelle capacité devez-vous activer ?

A. Password Hash Synchronization  
B. Password Writeback  
C. Seamless SSO  
D. Smart Lockout

### 15.
Un compte utilisateur est suspecté d'être compromis.

Vous voulez invalider ses sessions à l'aide de Microsoft Graph PowerShell.

Quelle commande est adaptée ?

A. `Remove-MgUser`  
B. `Reset-MgUserAuthenticationMethod`  
C. `Revoke-MgUserSignInSession`  
D. `Remove-MgServicePrincipal`

### 16.
Votre entreprise veut empêcher l'utilisation de mots de passe contenant le nom de sa société, ses marques et certains termes propres à son activité.

Quelle fonctionnalité répond directement à ce besoin ?

A. Conditional Access Authentication Strength  
B. Microsoft Entra Password Protection avec liste personnalisée  
C. SSPR Registration Campaign  
D. Identity Secure Score

### 17.
Une application contient une zone « données financières » beaucoup plus sensible que le reste de l'application.

Vous voulez exiger une authentification résistante au phishing **uniquement lors de l'accès à cette zone**, et pas pour toute l'application.

Que devez-vous utiliser ?

A. Named Location  
B. Access Package  
C. Conditional Access Authentication Context  
D. Application Collection

### 18.
Les administrateurs peuvent normalement modifier des stratégies Conditional Access.

Vous voulez qu'au moment précis où ils tentent de modifier ces stratégies, ils doivent utiliser une authentification plus forte.

Quelle fonctionnalité est la plus appropriée ?

A. PIM for Groups  
B. Protected Actions  
C. External Collaboration Settings  
D. Identity Secure Score

### 19.
Votre entreprise veut bloquer les protocoles d'authentification hérités qui ne prennent pas correctement en charge les mécanismes modernes.

Quelle partie d'une stratégie Conditional Access est particulièrement utile pour cibler ces connexions ?

A. Client apps / conditions d'application cliente  
B. User risk  
C. Authentication context  
D. Terms of Use

### 20.
Une stratégie Conditional Access comporte les contrôles suivants :

- Require multifactor authentication
- Require device to be marked as compliant

L'entreprise exige que **les deux conditions soient satisfaites**.

Comment configurez-vous les Grant Controls ?

A. Sélectionner uniquement MFA  
B. Require one of the selected controls  
C. Créer deux Named Locations  
D. Require all the selected controls

### 21.
Un utilisateur possède un access token encore valide pour SharePoint Online.

Son compte Microsoft Entra est ensuite désactivé.

Quel mécanisme peut permettre à une ressource compatible de cesser d'accepter le token avant son expiration normale ?

A. Continuous Access Evaluation  
B. Group expiration policy  
C. Password Hash Synchronization  
D. My Apps

### 22.
Une seule tentative de connexion provient d'une adresse IP anonyme et présente des caractéristiques inhabituelles.

Vous voulez prendre une décision d'accès en fonction du risque associé **à cette tentative précise**.

Quel signal devez-vous utiliser ?

A. User risk  
B. Sign-in risk  
C. Identity Secure Score  
D. Service principal risk uniquement

### 23.
Votre organisation veut pousser les utilisateurs qui utilisent encore des méthodes faibles à enregistrer Microsoft Authenticator.

Quelle fonctionnalité est prévue pour provoquer progressivement cette inscription ?

A. Access Reviews  
B. Company Branding  
C. Authentication Methods Registration Campaign  
D. Application Proxy

### 24.
Vous utilisez Global Secure Access et voulez réduire le risque de rejeu d'un token Microsoft 365 depuis une machine située hors de votre réseau protégé.

Vous voulez que Conditional Access puisse vérifier que le trafic passe bien par le service Global Secure Access de **votre tenant**.

Que devez-vous utiliser ?

A. Compliant network check  
B. Password Writeback  
C. Cross-tenant trust  
D. Device registration uniquement

### 25.
Un service Windows local s'exécute sur plusieurs serveurs joints au domaine AD DS.

Il doit utiliser un compte dont le mot de passe est automatiquement géré par Active Directory et utilisable sur plusieurs hôtes autorisés.

Quel type d'identité est le plus adapté ?

A. System-assigned Managed Identity  
B. User-assigned Managed Identity  
C. Compte utilisateur Microsoft Entra  
D. Group Managed Service Account (gMSA)

### 26.
Vous devez déléguer la gestion :

- des App Registrations ;
- des Enterprise Applications ;
- de Microsoft Entra Application Proxy.

Quel rôle est le moins privilégié répondant à **l'ensemble** de ces besoins ?

A. Cloud Application Administrator  
B. Application Administrator  
C. User Administrator  
D. Security Administrator

### 27.
Une Enterprise Application possède :

`Assignment required = Yes`

L'administrateur a déjà accordé tout le consentement nécessaire aux permissions API.

Bob n'est affecté ni directement ni par un groupe.

Que se passe-t-il ?

A. Bob ne peut pas accéder à l'application tant qu'il n'est pas affecté  
B. Le consentement admin lui donne automatiquement accès  
C. Bob obtient un rôle d'application par défaut  
D. `Assignment required` agit uniquement sur les administrateurs

### 28.
Les utilisateurs ne sont pas autorisés à consentir eux-mêmes à certaines permissions.

Vous voulez qu'ils puissent envoyer une demande aux administrateurs, qui pourront ensuite l'accepter ou la refuser.

Quelle fonctionnalité devez-vous configurer ?

A. PIM approval  
B. Access Package request  
C. Admin consent request workflow  
D. Access Review

### 29.
Vous examinez un access token Microsoft Graph et observez :

`scp = "User.Read Calendars.Read"`

Que pouvez-vous en déduire ?

A. Le token est nécessairement app-only  
B. Le token contient des permissions déléguées  
C. Le token provient d'une managed identity  
D. `scp` contient des app roles applicatifs

### 30.
Une application expose un rôle :

`Finance.Approver`

Vous affectez ce rôle à Alice.

Dans quel claim d'un token utilisateur destiné à cette application ce rôle peut-il apparaître ?

A. `aud`  
B. `scp` uniquement  
C. `roles`  
D. `iss`

### 31.
Une application SaaS développée par votre entreprise doit pouvoir être utilisée par des utilisateurs appartenant à **n'importe quel tenant Microsoft Entra d'une autre organisation**.

Quel type de compte pris en charge devez-vous choisir lors de l'App Registration ?

A. Accounts in this organizational directory only  
B. Accounts in any organizational directory  
C. Personal Microsoft accounts only  
D. Managed identities only

### 32.
Une application web est correctement enregistrée, mais lors de l'authentification Microsoft Entra retourne une erreur indiquant que l'URI de redirection ne correspond pas à celle enregistrée.

Quelle cause est la plus probable ?

A. Le service principal n'a pas de rôle Azure RBAC  
B. Le tenant utilise Security Defaults  
C. Le groupe utilisateur n'est pas dynamique  
D. La redirect URI envoyée ne correspond pas exactement à une URI configurée pour l'application

### 33.
Une entreprise possède 70 applications disponibles dans My Apps.

Elle veut afficher des groupes logiques tels que :

- RH
- Finance
- Développement

sans modifier les permissions des applications.

Que devez-vous utiliser ?

A. Application Collections  
B. Administrative Units  
C. Access Packages  
D. Connected Organizations

### 34.
Une application SaaS tierce prend en charge SCIM.

Vous voulez que Microsoft Entra crée, mette à jour et désactive automatiquement les utilisateurs dans cette application selon leurs affectations.

Que devez-vous configurer ?

A. Password-based SSO  
B. Automatic provisioning avec SCIM  
C. Conditional Access App Control  
D. Application Proxy

### 35.
Vous voulez que Defender for Cloud Apps obtienne des informations directement depuis une application SaaS prise en charge grâce à son API.

Quelle fonctionnalité utilisez-vous ?

A. Cloud Discovery snapshot report  
B. App connector / Connected app  
C. Global Secure Access client  
D. PIM resource audit

### 36.
Un utilisateur accède à une application SaaS depuis un appareil non géré.

Vous voulez :

- autoriser la connexion ;
- empêcher le téléchargement de fichiers sensibles pendant la session.

Quelles deux configurations sont nécessaires ?  
**Choisissez deux réponses.**

A. Une stratégie Conditional Access utilisant **Conditional Access App Control**  
B. Une policy de Password Protection  
C. Une session policy Defender for Cloud Apps  
D. Une cross-tenant synchronization  
E. Une managed identity

### 37.
Votre tenant a accordé de nombreux consentements OAuth à des applications tierces.

Vous voulez détecter et contrôler les applications OAuth qui présentent un comportement ou des permissions à risque.

Quelle capacité Defender for Cloud Apps est la plus adaptée ?

A. Cloud Discovery uniquement  
B. File policy  
C. Azure RBAC  
D. OAuth app policies

### 38.
Vous construisez un Access Package contenant :

- un groupe ;
- une application ;
- un site SharePoint.

Vous devez définir **qui peut demander cet accès, qui doit approuver la demande et pendant combien de temps l'accès reste valide**.

Quel objet configure ces règles ?

A. Catalog  
B. Connected Organization uniquement  
C. Access Package Policy  
D. Access Review

### 39.
Tous les consultants doivent accepter un document juridique avant d'accéder à une application sensible.

L'acceptation doit être vérifiée au moment de la connexion.

Quelle solution devez-vous utiliser ?

A. Company Branding  
B. Terms of Use ciblées par Conditional Access  
C. Application Collection  
D. Custom Security Attribute uniquement

### 40.
Un consultant externe a été invité **par Entitlement Management**.

Il perd sa dernière affectation d'Access Package et les paramètres de cycle de vie par défaut sont conservés.

Que se passe-t-il normalement ?

A. Son compte devient automatiquement Member permanent  
B. Rien ne change jamais sur son compte Guest  
C. Son compte est supprimé immédiatement sans blocage préalable  
D. Il est bloqué de connexion puis son compte peut être supprimé après la période configurée, 30 jours par défaut

### 41.
Une Access Review est terminée.

Les reviewers ont choisi **Deny** pour plusieurs utilisateurs, mais `Auto apply results to resource` n'était pas activé.

Que doit faire l'administrateur ?

A. Appliquer manuellement les résultats de la review  
B. Créer un Access Package  
C. Attendre une synchronisation PHS  
D. Activer PIM for Groups

### 42.
Un ingénieur doit pouvoir activer temporairement le rôle Azure **Contributor** au niveau d'un abonnement.

Quelle zone de PIM devez-vous utiliser ?

A. Microsoft Entra roles  
B. Groups  
C. Azure resources  
D. Authentication methods

### 43.
Le groupe `GRP-Sensitive` donne accès à une application critique.

Alice doit pouvoir devenir **propriétaire du groupe uniquement à la demande**, pendant une durée limitée.

Quelle configuration devez-vous utiliser ?

A. Dynamic membership  
B. PIM for Groups avec une affectation eligible Owner  
C. Access Review avec Alice comme reviewer  
D. Group-based licensing

### 44.
Une affectation PIM est éligible.

Chaque activation doit nécessiter l'approbation du responsable sécurité.

Où configurez-vous cette exigence ?

A. Dans les paramètres d'activation du rôle PIM  
B. Dans Company Branding  
C. Dans le service principal de l'application  
D. Dans le Password Protection agent

### 45.
Vous concevez les comptes d'accès d'urgence de votre tenant.

Quelles pratiques sont appropriées ?  
**Choisissez trois réponses.**

A. Utiliser des comptes cloud-only indépendants de l'infrastructure locale  
B. Les exclure des stratégies Conditional Access susceptibles de verrouiller le tenant  
C. Les utiliser quotidiennement comme comptes administrateurs principaux  
D. Maintenir des comptes d'urgence disposant des privilèges nécessaires pour reprendre le contrôle  
E. Les synchroniser obligatoirement depuis AD DS

### 46.
Vous configurez les Diagnostic Settings de Microsoft Entra afin de conserver et exploiter les journaux au-delà de leur usage immédiat.

Quelles destinations sont prises en charge ?  
**Choisissez trois réponses.**

A. Log Analytics workspace  
B. Storage account  
C. Event Hub  
D. Windows Registry  
E. Local Event Viewer uniquement

### 47.
Vous voulez analyser dans Log Analytics les connexions Microsoft Entra ayant échoué.

Quelle table KQL constitue le point de départ le plus logique ?

A. `SigninLogs`  
B. `AADProvisioningLogs`  
C. `AzureActivity` uniquement  
D. `SecurityEvent` uniquement

### 48.
Votre responsable sécurité veut disposer d'un tableau graphique montrant les tendances d'authentification et de Conditional Access sans écrire à chaque fois de nouvelles requêtes.

Quel outil Microsoft Entra est conçu pour ce type d'analyse visuelle ?

A. Access Packages  
B. PIM for Groups  
C. Workbooks  
D. Application Collections

### 49.
Le responsable IAM demande une mesure permettant d'identifier des recommandations pour améliorer la posture de sécurité des identités.

Il ne veut pas d'une fonctionnalité qui bloque directement les connexions.

Que devez-vous utiliser ?

A. Sign-in risk  
B. Identity Secure Score  
C. Conditional Access Block  
D. Smart Lockout

### 50.
Vous recevez un fichier CSV contenant 500 nouveaux employés.

Vous devez créer leurs comptes Microsoft Entra automatiquement avec Microsoft Graph PowerShell.

Quelle approche est la plus adaptée ?

A. `Import-Csv` puis création des comptes avec `New-MgUser`  
B. Créer 500 invitations B2B avec `New-MgInvitation`  
C. Créer un Access Package par utilisateur  
D. Utiliser Microsoft Entra Application Proxy

---

<details>
<summary><strong>Corrigé</strong></summary>

| # | Réponse | Point vérifié |
|---:|:---:|---|
| 1 | B | Attribution de valeurs de custom security attributes |
| 2 | B | Cross-tenant sync crée par défaut des External Members |
| 3 | A | Company Branding |
| 4 | C | BYOD : Microsoft Entra registered |
| 5 | B | Cloud Sync pour forêts déconnectées |
| 6 | A | Connect Sync pour le scénario classique Hybrid Entra Join attendu au SC-300 |
| 7 | D | `usageLocation` nécessaire à l'attribution de licence |
| 8 | C | Connect Health |
| 9 | A | Fédération directe / IdP externe SAML |
| 10 | B | Permissions effectives = combinaison des affectations selon leur portée |
| 11 | A | `X509SKI` et CBA |
| 12 | B | Temporary Access Pass |
| 13 | A, C, D | Méthodes phishing-resistant |
| 14 | B | Password Writeback |
| 15 | C | `Revoke-MgUserSignInSession` |
| 16 | B | Entra Password Protection |
| 17 | C | Authentication Context |
| 18 | B | Protected Actions |
| 19 | A | Blocage de legacy authentication via Client apps |
| 20 | D | Require all selected controls |
| 21 | A | Continuous Access Evaluation |
| 22 | B | Sign-in risk = tentative précise |
| 23 | C | Registration Campaign |
| 24 | A | Compliant network check |
| 25 | D | gMSA pour service Windows multi-hôtes AD DS |
| 26 | B | Application Administrator gère aussi Application Proxy |
| 27 | A | Assignment required reste indépendant du consentement |
| 28 | C | Admin consent request workflow |
| 29 | B | `scp` = permissions déléguées |
| 30 | C | App roles dans `roles` |
| 31 | B | Application multitenant Entra |
| 32 | D | Redirect URI doit correspondre exactement |
| 33 | A | Application Collections |
| 34 | B | Provisioning SCIM |
| 35 | B | Connected apps / app connectors |
| 36 | A, C | CA App Control + session policy Defender for Cloud Apps |
| 37 | D | OAuth app policies |
| 38 | C | Access Package Policy |
| 39 | B | Terms of Use + Conditional Access |
| 40 | D | Lifecycle des externes Entitlement Management |
| 41 | A | Résultats d'Access Review à appliquer manuellement |
| 42 | C | PIM for Azure resources |
| 43 | B | PIM for Groups, Owner éligible |
| 44 | A | PIM role settings / activation settings |
| 45 | A, B, D | Comptes emergency access |
| 46 | A, B, C | Destinations Diagnostic Settings |
| 47 | A | `SigninLogs` |
| 48 | C | Workbooks |
| 49 | B | Identity Secure Score |
| 50 | A | Bulk création via CSV + Microsoft Graph PowerShell |

### Résultat de la session de validation

Lors de la session de préparation du 24 août 2026, ce test a donné **46/50 (92 %)**.

Les quatre points ayant nécessité une correction étaient :

- Cross-tenant synchronization : **External Member** par défaut ;
- blocage de l'authentification héritée : **Client apps** dans Conditional Access ;
- service Windows local multi-hôtes avec mot de passe géré par AD DS : **gMSA** ;
- applications OAuth tierces à risque : **OAuth app policies**, et non Cloud Discovery.

</details>
