# Identités utilisateurs

Domaine « Implement and manage user identities », 20 à 25 % de l'examen.

Ce domaine couvre le tenant, les rôles, les comptes, groupes, appareils, identités externes et identité hybride. Les questions se jouent souvent sur la **portée**, le **type d'objet**, le **mécanisme d'attribution** et la **licence**.

## Tenant, abonnement et périmètre d'administration

Un tenant Microsoft Entra est une instance d'annuaire. Un abonnement Azure est un conteneur de ressources et de facturation. Un abonnement fait confiance à un tenant pour l'authentification, mais un tenant peut servir plusieurs abonnements ou exister sans abonnement Azure.

Deux systèmes d'autorisation sont à distinguer :

| | Rôles Microsoft Entra | Azure RBAC |
|---|---|---|
| Objet gouverné | annuaire : utilisateurs, groupes, applications, rôles | ressources Azure |
| Portée | tenant, administrative unit, application selon rôle | management group, abonnement, resource group, ressource |
| Exemple | Global Administrator, User Administrator | Owner, Contributor, Reader |

Un Global Administrator n'est pas automatiquement Owner d'un abonnement Azure. Le réglage **Access management for Azure resources** lui permet d'obtenir **User Access Administrator** à la portée racine `/` afin de reprendre la main sur les affectations Azure RBAC.

### Rôles à connaître

L'examen demande surtout le rôle le moins privilégié qui suffit.

| Besoin | Rôle attendu |
|---|---|
| Gérer les app registrations et enterprise applications | Application Administrator ou Cloud Application Administrator selon besoin |
| Consentir aux app roles Microsoft Graph | Privileged Role Administrator ou Global Administrator |
| Créer/supprimer utilisateurs, réinitialiser mots de passe des non-admins | User Administrator |
| Gérer les méthodes d'authentification d'utilisateurs non-admins / TAP | Authentication Administrator |
| Gérer les méthodes d'authentification de comptes administrateurs | Privileged Authentication Administrator |
| Lire les journaux sans modification | Reports Reader ou Security Reader |
| Gérer les affectations de rôles Microsoft Entra | Privileged Role Administrator |

Cloud Application Administrator ne couvre pas certaines fonctions comme Application Proxy, contrairement à Application Administrator.

## Administrative units

Une administrative unit limite la portée d'un rôle Entra à un sous-ensemble d'objets.

Une AU peut contenir utilisateurs, groupes et appareils, mais **l'appartenance n'est pas transitive**. Mettre un groupe dans une AU permet de gérer ce groupe dans la portée, pas automatiquement tous ses membres utilisateurs.

L'appartenance peut être assignée ou dynamique selon les fonctionnalités/licences disponibles.

Les **restricted management administrative units** protègent certains objets contre les rôles administratifs à portée tenant qui ne sont pas explicitement autorisés sur cette AU, avec des exceptions pour les rôles les plus élevés selon le produit.

## Domaines personnalisés

Tout tenant possède un domaine initial `*.onmicrosoft.com`.

Pour utiliser un domaine comme `contoso.com` :

1. ajouter le domaine au tenant ;
2. prouver sa propriété dans le DNS public ;
3. utiliser ensuite le suffixe vérifié pour les identités et services pris en charge.

Pour la vérification, les réponses d'examen classiques sont **TXT** ou **MX**.

À retenir :

```text
Ajouter le domaine
!=
Vérifier le domaine
```

Le domaine `onmicrosoft.com` initial reste une dépendance importante du tenant.

## Company Branding

Company Branding personnalise l'expérience de connexion : logo, image, texte et variantes localisées.

Ce réglage ne donne aucun accès et ne remplace pas Conditional Access. Si le besoin est « personnaliser l'écran de connexion », la réponse est Company Branding.

## Tenant, user, group et device settings

Le study guide demande aussi de savoir configurer les propriétés et paramètres du tenant, des utilisateurs, groupes et appareils.

Le raisonnement attendu est de savoir **à quel niveau vit le réglage** :

- propriétés générales du tenant ;
- politiques utilisateurs et consentement ;
- paramètres de groupe ;
- paramètres d'enregistrement / jonction des appareils.

## Utilisateurs

Un utilisateur peut être cloud-only, synchronisé depuis AD DS ou créé comme identité externe B2B.

La propriété `onPremisesSyncEnabled` aide à distinguer un objet synchronisé. Pour un objet dont la source d'autorité reste AD DS, certaines propriétés doivent être modifiées à la source puis synchronisées.

### Member et Guest

`UserType` vaut `Member` ou `Guest`. Cette propriété décrit la **relation à l'organisation**, pas la provenance technique de l'identité.

Accepter une invitation B2B fait passer l'état externe de `PendingAcceptance` à `Accepted`, mais ne transforme pas automatiquement un Guest en Member.

Il existe aussi des cas d'external members, notamment avec la cross-tenant synchronization.

## Custom security attributes

Les **custom security attributes** permettent de définir des attributs métier personnalisés organisés en attribute sets, puis de les affecter aux objets pris en charge.

Ils servent à porter des métadonnées de sécurité / métier réutilisables dans des scénarios d'autorisation et de gouvernance.

Ne pas les confondre avec :

- une extension de schéma AD DS ;
- un simple champ libre ajouté arbitrairement à un utilisateur ;
- les attributs dynamiques de groupe eux-mêmes.

## Opérations en masse

Microsoft Entra permet des opérations bulk dans le portail et via PowerShell / Microsoft Graph.

Pour les invitations B2B en masse, deux noms de colonnes à reconnaître :

```text
inviteeEmail
inviteRedirectUrl
```

L'examen peut demander de choisir entre opération individuelle et bulk, ou de reconnaître le fichier attendu.

## Licences utilisateur

Une licence peut être attribuée directement ou via un groupe.

La **group-based licensing** exige les prérequis de licence appropriés et l'utilisateur doit notamment avoir un `usageLocation` valide pour de nombreux services.

Une licence héritée d'un groupe se gère via le mécanisme qui l'a fournie ; on ne traite pas l'héritage comme une simple affectation individuelle.

En Graph, `GET /subscribedSkus` permet de récupérer les SKU et `skuId` nécessaires avant une affectation programmatique de licence.

## Groupes

| | Security group | Microsoft 365 group |
|---|---|---|
| Usage principal | autorisation, licences, affectations | collaboration Teams/SharePoint/M365 |
| Membres | utilisateurs, appareils, service principals, groupes selon scénario | utilisateurs et collaboration M365 |
| Appartenance | assignée ou dynamique selon type | assignée ou dynamique selon type |

### Appartenance assignée ou dynamique

Une appartenance assignée est gérée manuellement.

Une appartenance dynamique est calculée à partir d'une règle d'attributs utilisateur **ou** appareil.

Exemples :

```powershell
user.department -eq "Finance"
```

ou une règle sur `device.*`.

### Groupes role-assignable

Pour porter un rôle Microsoft Entra, le groupe doit être conçu comme **role-assignable** avec `isAssignableToRole`.

À retenir :

- la propriété se décide à la création ;
- le groupe ne doit pas être dynamique ;
- la gouvernance de ses membres/owners est particulièrement sensible.

### Owner et Member

Owner = gère le groupe.

Member = appartient au groupe et reçoit les entitlements distribués par cette appartenance.

Un Owner qui n'est pas Member ne reçoit pas automatiquement les licences, rôles Azure ou accès applicatifs du groupe.

## Appareils : registered, joined, hybrid joined

| État | Scénario typique |
|---|---|
| Microsoft Entra registered | BYOD / appareil personnel |
| Microsoft Entra joined | appareil d'entreprise cloud-first, sans join AD DS local |
| Hybrid Microsoft Entra joined | appareil joint à AD DS et enregistré dans Entra |

`Managed` et `Compliant` sont distincts : un appareil peut être géré mais non conforme.

Le Hybrid Microsoft Entra join dépend d'une synchronisation des appareils, ce qui oriente vers Connect Sync dans les scénarios hybrides correspondants.

## Identités externes

Quatre mécanismes à distinguer.

### External collaboration settings

Règles globales B2B : qui peut inviter, restrictions de lecture des invités, domaines autorisés/bloqués, etc.

### Cross-tenant access settings

Relation avec un tenant partenaire identifié : accès entrant/sortant et **trust settings** pour accepter notamment MFA ou signaux d'appareil du tenant partenaire.

### Cross-tenant synchronization

Provisioning automatique de comptes entre tenants, utile lorsque les utilisateurs d'une autre organisation/tenant doivent apparaître automatiquement dans le tenant cible.

### External identity providers

Permet aux externes de s'authentifier via des fournisseurs ou protocoles pris en charge, notamment SAML / WS-Fed selon le scénario, ou email one-time passcode pour certains invités.

## Identité hybride

### Connect Sync vs Cloud Sync

| | Connect Sync | Cloud Sync |
|---|---|---|
| Installation | moteur complet sur serveur Windows | agents légers + config cloud |
| Forêts déconnectées | moins adapté / contraintes de topologie | scénario fort de Cloud Sync |
| Synchronisation appareils / Hybrid join | Oui | Non |
| Scénarios Exchange hybrides complets | Oui | plus limité |
| Configuration | assistant local | portail Entra |

Le discriminant n'est pas « multi-forêts » à lui seul. Regarder : forêts déconnectées, appareils, Exchange hybride, fonctions de synchronisation nécessaires.

### Password Hash Synchronization

PHS synchronise vers Entra un dérivé du hash du mot de passe. L'authentification cloud ne dépend alors pas d'une validation AD DS en temps réel.

Avantages : résilience cloud et compatibilité avec certains signaux comme la détection d'identifiants divulgués.

### Pass-through Authentication

PTA valide le mot de passe contre AD DS au moment de la connexion via des agents.

Le mot de passe n'est pas validé par un hash cloud, mais l'authentification dépend alors de la disponibilité des agents et des DC.

### Federation

AD FS ou un fournisseur fédéré prend en charge l'authentification. C'est plus complexe et doit répondre à un besoin réel, pas être conservé par inertie.

### Seamless SSO

Ajoute une expérience silencieuse pour les postes AD DS joints dans les scénarios PHS/PTA pris en charge.

Les appareils Entra joined / hybrid joined modernes s'appuient notamment sur le PRT pour le SSO cloud, donc ne pas appliquer le raccourci Seamless SSO à tous les appareils.

### Password writeback

Répercute vers AD DS les changements/réinitialisations cloud dans les scénarios hybrides pris en charge.

### Migrer depuis AD FS

Le study guide actuel demande explicitement de savoir **migrer depuis AD FS**.

Le raisonnement :

- inventorier domaines fédérés et dépendances ;
- préparer une méthode cloud comme PHS ou PTA ;
- tester avant bascule ;
- utiliser PHS comme mécanisme de résilience quand pertinent ;
- réduire la dépendance à AD FS sauf besoin fonctionnel justifié.

Réflexe scénario :

- moins de dépendance on-prem -> PHS ;
- validation du mot de passe contre AD DS à chaque sign-in sans AD FS -> PTA ;
- SSO silencieux sur postes AD DS joints -> Seamless SSO en complément si pertinent.

### Connect Health

Microsoft Entra Connect Health surveille les composants hybrides pris en charge et remonte erreurs / alertes de synchronisation ou d'infrastructure selon la configuration.

## Ce qui se joue sur des détails

- Tenant != abonnement Azure.
- Global Administrator != Owner Azure.
- Administrative Unit != Security Group ; AU non transitive via les groupes.
- Custom domain : **TXT/MX** pour la vérification.
- Company Branding = expérience de connexion.
- Bulk B2B : `inviteeEmail` et `inviteRedirectUrl`.
- Custom security attributes = métadonnées personnalisées gérées, pas extension AD DS.
- Owner de groupe != Member.
- Role-assignable group != dynamic group.
- Registered != Joined != Hybrid Joined.
- Cross-tenant access != cross-tenant synchronization.
- PHS != PTA.
- Hybrid join -> besoin de synchronisation appareil -> Connect Sync.
- Migration AD FS : privilégier une méthode plus simple lorsque le besoin ne justifie plus la fédération.
