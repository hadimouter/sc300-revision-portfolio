# Glossaire

Les termes sont regroupés par domaine plutôt que par ordre alphabétique, parce que la difficulté vient rarement d'un mot isolé et presque toujours de sa place dans un ensemble.

## Annuaire et administration

**Tenant** : instance d'annuaire Microsoft Entra dédiée à une organisation. Périmètre de sécurité et d'administration. Un abonnement Azure fait confiance à un seul tenant ; un tenant peut servir plusieurs abonnements ou aucun.

**Microsoft Entra** : nom de la famille de produits, qui comprend Microsoft Entra ID, Microsoft Entra ID Governance, Microsoft Entra Permissions Management, Global Secure Access et Microsoft Entra Verified ID.

**Microsoft Entra ID** : le service d'annuaire et d'identité proprement dit, anciennement Azure Active Directory. C'est de lui que traite l'essentiel du SC-300.

**Rôle Microsoft Entra** : rôle d'administration portant sur les objets d'annuaire. Sa portée peut être le tenant, une administrative unit, ou une application.

**Azure RBAC** : système d'autorisation d'Azure Resource Manager, portant sur les ressources Azure. Une affectation associe un principal, un rôle et une portée (management group, abonnement, resource group, ressource).

**Administrative unit** : conteneur d'objets d'annuaire permettant de restreindre la portée d'un rôle Entra à un sous-ensemble. L'appartenance n'est pas transitive : un groupe placé dans une AU n'y place pas ses membres.

**Restricted management administrative unit** : administrative unit dont les objets sont protégés contre les administrateurs à portée tenant. Seuls les administrateurs explicitement affectés à l'AU, et le Global Administrator, peuvent agir dessus.

## Utilisateurs, groupes, appareils

**UserType** : propriété valant `Member` ou `Guest`, décrivant la relation à l'organisation. Indépendante de l'endroit où l'utilisateur s'authentifie : il existe des internal guests et des external members.

**External member** : utilisateur qui s'authentifie auprès d'un autre tenant mais est traité comme un membre. C'est le résultat par défaut de la cross-tenant synchronization.

**Groupe assigné** : groupe dont l'appartenance est gérée manuellement.

**Groupe dynamique** : groupe dont l'appartenance est calculée par une règle sur des attributs utilisateur ou appareil. Exige P1 pour chaque membre.

**Groupe role-assignable** : groupe pouvant porter un rôle Microsoft Entra. La propriété `isAssignableToRole` se fixe à la création et ne peut plus changer ; le groupe ne peut pas être dynamique.

**Group-based licensing** : attribution de licences par appartenance à un groupe. Exige P1 et un `usageLocation` renseigné sur chaque utilisateur.

**Microsoft Entra registered** : appareil enregistré dans Entra sans y être joint. Scénario personnel ou BYOD.

**Microsoft Entra joined** : appareil appartenant à l'organisation et joint directement à Entra, sans Active Directory local.

**Hybrid Microsoft Entra joined** : appareil joint à un Active Directory local et enregistré dans Entra. Exige la synchronisation des appareils, donc Connect Sync.

**Managed** : appareil géré par une solution de gestion, typiquement Intune.

**Compliant** : appareil satisfaisant les règles d'une compliance policy. Un appareil peut être managed sans être compliant.

**Primary Refresh Token (PRT)** : jeton délivré à un appareil joint ou enregistré, qui porte le SSO de l'utilisateur sur cet appareil.

## Applications et identités de workload

**Application object** : définition globale d'une application, dans son tenant d'origine. Visible sous App registrations.

**Service principal** : instance locale d'une application dans un tenant, porteuse des consentements et affectations locales. Visible sous Enterprise applications. Même Client ID que l'application object, Object ID différent.

**Application (client) ID** : identifiant de l'application, commun à l'application object et à tous ses service principals.

**Permission déléguée** : permission utilisée au nom d'un utilisateur connecté. Représentée par un `oauth2PermissionScope`, exposée dans le claim `scp`. Droits effectifs égaux à l'intersection des permissions de l'application et de ceux de l'utilisateur.

**Permission applicative** : permission utilisée par l'application avec sa propre identité, sans utilisateur. Représentée par un `appRole`, exposée dans le claim `roles`. Exige toujours un consentement administrateur.

**App role** : rôle applicatif défini dans le manifeste d'une application. Assignable à des utilisateurs, à des groupes ou à des applications selon `allowedMemberTypes`. Apparaît dans le claim `roles`, y compris dans un token utilisateur.

**Admin consent** : consentement accordé par un administrateur au nom de l'organisation. Sur des permissions déléguées, il produit un `oauth2PermissionGrant` avec `consentType` valant `AllPrincipals`.

**Permission admin-restricted** : permission déléguée qu'un utilisateur ne peut jamais consentir lui-même, comme `User.Read.All` ou `Directory.ReadWrite.All`.

**Client credentials flow** : flux OAuth 2.0 d'authentification d'une application sans utilisateur. Exige le scope `.default`.

**`.default`** : scope signifiant « toutes les permissions déjà configurées et consenties pour cette ressource ». Obligatoire dans le client credentials flow.

**`idtyp`** : claim valant `app` dans un token app-only. Marqueur fiable, contrairement à la simple présence de `roles`.

**Managed identity** : identité de workload dont Azure gère les identifiants. Représentée par un service principal, sans application object ni credential manipulable. System-assigned si liée au cycle de vie d'une ressource, user-assigned si autonome et partageable.

**Workload identity federation** : mécanisme permettant à un workload hors d'Azure d'échanger un jeton de son propre IdP contre un token Entra, sans stocker de secret.

**Assignment required** : réglage d'une enterprise application limitant l'obtention d'un token aux principals explicitement affectés. N'accorde aucun consentement. L'affectation par groupe exige P1.

**Application Proxy** : publication d'une application web interne vers l'extérieur via un private network connector sortant. Exige P1. La préauthentification Entra est requise pour appliquer le Conditional Access.

**Private network connector** : agent installé sur le réseau interne, n'ouvrant que des connexions sortantes. Commun au proxy d'application et à Microsoft Entra Private Access.

## Authentification

**Authentication methods policy** : configuration de ce que les utilisateurs ont le droit d'utiliser pour s'authentifier.

**Authentication strength** : ensemble nommé de combinaisons de méthodes considérées comme suffisantes dans un contexte donné. Trois valeurs intégrées : MFA, Passwordless MFA, Phishing-resistant MFA. Exclusive avec « Require multifactor authentication » dans une même policy CA.

**Temporary Access Pass (TAP)** : code temporaire créé par un administrateur pour un autre utilisateur, servant à l'amorçage ou à la récupération. Le caractère à usage unique se décide à la création de chaque pass.

**Passkey FIDO2** : identifiant résistant au phishing, matériel ou stocké dans Microsoft Authenticator.

**Number matching** : obligation de saisir dans Microsoft Authenticator un nombre affiché à l'écran, pour contrer la fatigue de notification.

**SSPR** : self-service password reset. Exige le password writeback en environnement hybride.

**Microsoft Entra Password Protection** : blocage des mots de passe faibles à partir d'une liste globale et d'une liste personnalisée d'au plus 1 000 termes, applicable aussi à l'Active Directory local via un agent.

## Conditional Access et risque

**Conditional Access** : moteur de décision évaluant des signaux à chaque connexion pour autoriser, renforcer ou bloquer.

**Assignments** : bloc de ciblage d'une policy CA, comprenant Users, Target resources, Network et Conditions.

**Grant control** : exigence à satisfaire pour que l'accès soit accordé. La MFA en fait partie.

**Session control** : contrôle appliqué après l'octroi, dont sign-in frequency, persistent browser session, continuous access evaluation et Conditional Access App Control.

**Continuous access evaluation (CAE)** : révocation quasi immédiate d'une session en cours sur événement critique, sans attendre l'expiration du token.

**Authentication context** : étiquette permettant d'appliquer une policy CA à une opération précise plutôt qu'à une application entière.

**Protected action** : opération d'administration Entra liée à un authentication context, exigeant une authentification renforcée au moment de l'exécution.

**Named location** : plage d'adresses IP ou liste de pays définie par l'administrateur et utilisable dans le bloc Network d'une policy CA. À distinguer de l'emplacement observé dans un log, qui est une donnée et non un objet de configuration.

**Report-only** : mode d'évaluation d'une policy sans application, destiné à mesurer l'impact avant activation.

**What If** : simulation du résultat de policies CA à partir de signaux fournis manuellement.

**Security Defaults** : protection de base gratuite imposant la MFA à tous. Techniquement incompatible avec le Conditional Access : l'un exclut l'autre.

**Microsoft Entra ID Protection** : service de détection et de remédiation du risque d'identité. Nom actuel de l'ancien Azure AD Identity Protection.

**User risk** : probabilité que le compte lui-même soit compromis.

**Sign-in risk** : probabilité qu'une tentative de connexion précise soit illégitime.

**Risk detection** : événement individuel alimentant un score de risque.

## Gouvernance

**Identity governance** : ensemble des mécanismes répondant à pourquoi un accès existe, pour combien de temps, qui l'approuve et qui le recertifie.

**Catalog** : conteneur de ressources utilisables dans des access packages. N'accorde rien par lui-même.

**Access package** : ensemble de ressources (groupes, applications, sites SharePoint) demandable comme un tout.

**Resource role** : rôle dans une ressource incluse dans un access package. Member ou Owner pour un groupe, un app role pour une application, un niveau d'accès pour un site.

**Policy (Entitlement Management)** : définition du workflow de demande : qui peut demander, qui approuve, quelles informations sont exigées, quelle durée.

**Connected organization** : représentation d'un partenaire externe, dont l'identité peut provenir d'un tenant Entra, d'un IdP fédéré, d'un domaine de messagerie ou d'un tenant B2C.

**My Access** : portail utilisateur (`myaccess.microsoft.com`) pour demander des access packages et répondre aux access reviews.

**Access review** : recertification périodique d'un accès existant. Ne sert jamais à attribuer un accès.

**Auto apply results to resource** : réglage déterminant si les décisions d'une access review sont appliquées automatiquement à sa clôture.

**PIM** : Privileged Identity Management. Gouverne les rôles Microsoft Entra, les rôles Azure resource et les groupes.

**PIM for Groups** : éligibilité à devenir membre ou propriétaire d'un groupe, activable à la demande. Seule voie pour rendre temporaire un accès distribué par groupe.

**Éligible** : affectation activable à la demande, ne conférant aucun privilège tant qu'elle n'est pas activée.

**Active** : affectation conférant les privilèges immédiatement.

**Lifecycle Workflows** : automatisation des tâches Joiner, Mover, Leaver, déclenchée par `employeeHireDate` et `employeeLeaveDateTime`.

**Break-glass account** : compte d'urgence cloud-only, Global Administrator permanent hors PIM, servant à reprendre la main en cas de verrouillage du tenant.

## Identités externes et hybride

**External collaboration settings** : règles générales d'invitation B2B, restrictions de lecture d'annuaire des invités, domaines autorisés ou bloqués.

**Cross-tenant access settings** : relation d'accès entrant et sortant avec un tenant partenaire, incluant les trust settings qui permettent d'accepter la MFA ou la conformité d'appareil effectuée chez le partenaire.

**Cross-tenant synchronization** : provisionnement automatique d'utilisateurs entre tenants, produisant par défaut des external members.

**Email one-time passcode** : méthode de repli permettant à un invité sans compte Entra ni compte Microsoft de s'authentifier par un code reçu par courriel.

**Entra Connect Sync** : moteur de synchronisation hybride complet, installé sur un serveur. Seul à gérer les appareils et donc le hybrid join.

**Cloud Sync** : synchronisation par agents légers, configurée dans le portail. Seule à gérer les forêts déconnectées. Ne synchronise pas les appareils.

**PHS** : Password Hash Synchronization. Synchronise un hash du hash du mot de passe, l'authentification se faisant dans le cloud.

**PTA** : Pass-through Authentication. Valide le mot de passe contre l'Active Directory local via des agents sortants.

**Seamless SSO** : connexion silencieuse par Kerberos pour les appareils joints au domaine Active Directory. Ne concerne pas les appareils Entra joined ou hybrid joined, qui utilisent leur PRT.

**Password writeback** : répercussion vers l'Active Directory local des réinitialisations effectuées dans le cloud. Exige P1.

**Connect Health** : supervision des agents de synchronisation, serveurs AD FS et contrôleurs de domaine. Exige P1.

## Accès réseau et applications SaaS

**Global Secure Access** : offre Security Service Edge de Microsoft Entra, regroupant Private Access et Internet Access.

**Microsoft Entra Private Access** : accès Zero Trust à des applications privées, application par application, tous protocoles TCP et UDP, sans exposer le réseau ni publier d'URL.

**Microsoft Entra Internet Access** : sécurisation du trafic sortant vers Internet, avec filtrage web et application du Conditional Access au trafic réseau.

**Compliant network check** : contrôle Conditional Access vérifiant que la connexion transite par le réseau Microsoft, contrant le rejeu de token depuis ailleurs.

**Defender for Cloud Apps** : visibilité sur l'usage des applications cloud et contrôle en temps réel des sessions.

**Access policy (Defender for Cloud Apps)** : autorise ou bloque l'accès à une application.

**Session policy (Defender for Cloud Apps)** : contrôle le comportement pendant la session, dont téléchargement, copie et impression.

**Conditional Access App Control** : session control d'une policy CA routant l'application vers le reverse proxy de Defender for Cloud Apps. Prérequis commun aux access policies et aux session policies.

## Journaux

**Sign-in logs** : journal des authentifications, en quatre catégories : interactive, non-interactive, service principal, managed identity.

**Audit logs** : journal des modifications de configuration de l'annuaire.

**Provisioning logs** : journal des opérations du moteur de provisioning vers une application cible.

**Request ID** : identifiant d'une requête unique.

**Correlation ID** : identifiant regroupant les requêtes d'une même opération.

**Diagnostic settings** : configuration d'export des logs vers un Log Analytics workspace, un storage account ou un Event Hub.

**Identity Secure Score** : indicateur de posture d'identité. Ne bloque aucun accès.

**SCIM** : System for Cross-domain Identity Management, standard de provisioning d'identités vers des applications.

## Concepts transverses

**RBAC** : autorisation fondée sur des rôles attribués à des principals.

**ABAC** : autorisation fondée sur des attributs et des conditions évaluées au moment de l'accès. Dans Azure, les conditions de role assignment sur le stockage en sont une mise en œuvre.

**IdP** : fournisseur d'identité, qui authentifie et émet des jetons.

**IGA** : Identity Governance and Administration, couche de gouvernance du cycle de vie des accès.

**Zero Trust** : modèle supposant la compromission et exigeant vérification explicite, moindre privilège et limitation du rayon d'action.
