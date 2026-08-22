# Identités utilisateurs

Domaine « Implement and manage user identities », 20 à 25 % de l'examen.

Ce domaine couvre le socle de l'annuaire : le tenant lui-même, les rôles d'administration, les comptes, les groupes, les identités externes et la synchronisation avec un Active Directory local. C'est le domaine le plus large en surface et le moins profond en difficulté, mais c'est aussi celui où les questions se jouent sur des détails de portée : qui peut faire quoi, sur quels objets, avec quelle licence.

## Tenant, abonnement et périmètre d'administration

Un tenant Microsoft Entra est une instance d'annuaire. Un abonnement Azure est un conteneur de facturation pour des ressources. Un abonnement fait confiance à exactement un tenant pour authentifier ses utilisateurs, mais un tenant peut servir plusieurs abonnements, et un tenant peut parfaitement exister sans le moindre abonnement Azure.

Cette séparation se traduit par deux systèmes d'autorisation qui ne communiquent pas :

| | Rôles Microsoft Entra | Azure RBAC |
|---|---|---|
| Objet gouverné | Objets d'annuaire : utilisateurs, groupes, applications, rôles | Ressources Azure : machines virtuelles, stockage, réseaux |
| Portée possible | Tenant entier, ou une administrative unit, ou une application | Management group, abonnement, resource group, ressource |
| Stockage | Dans l'annuaire Entra | Dans Azure Resource Manager |
| Exemple | Global Administrator, User Administrator | Owner, Contributor, Reader |

Un Global Administrator n'est donc pas Owner des abonnements Azure du tenant. La nuance que l'examen exploite est qu'il peut le devenir : dans Microsoft Entra ID puis Properties, le commutateur **Access management for Azure resources** lui attribue le rôle Azure RBAC **User Access Administrator** à la portée racine `/`. Ce n'est ni automatique, ni le rôle Owner, ni permanent, et l'opération est tracée dans les Audit logs. Un scénario qui demande « comment un Global Administrator peut-il reprendre la main sur un abonnement orphelin » attend cette réponse.

### Rôles à connaître

L'examen ne demande pas de réciter les 100 rôles intégrés, mais de choisir le moins privilégié qui suffit.

| Besoin | Rôle attendu |
|---|---|
| Gérer tous les aspects des applications, y compris consentir aux app roles Microsoft Graph | Privileged Role Administrator |
| Gérer les app registrations et enterprise applications, consentir à toutes les API sauf les app roles Graph | Application Administrator ou Cloud Application Administrator |
| Créer et supprimer des utilisateurs, réinitialiser les mots de passe des non-administrateurs | User Administrator |
| Gérer les méthodes d'authentification des utilisateurs non-administrateurs, générer un Temporary Access Pass | Authentication Administrator |
| Faire la même chose sur des comptes administrateurs | Privileged Authentication Administrator |
| Lire les journaux de connexion et d'audit sans rien modifier | Reports Reader ou Security Reader |
| Attribuer des rôles à d'autres utilisateurs | Privileged Role Administrator |

La distinction Application Administrator contre Cloud Application Administrator tient à un seul point : Cloud Application Administrator ne peut pas gérer le proxy d'application. Tout le reste est identique.

### Administrative units

Une administrative unit est un conteneur qui restreint la portée d'un rôle Entra à un sous-ensemble d'objets. Un User Administrator porté par une AU ne peut administrer que les utilisateurs, groupes ou appareils placés dans cette AU.

Trois points reviennent régulièrement :

Une AU peut contenir des utilisateurs, des groupes et des appareils, mais l'appartenance n'est pas transitive. Placer un groupe dans une AU donne à l'administrateur de l'AU le droit de gérer ce groupe, pas ses membres. Pour administrer les utilisateurs, il faut les ajouter individuellement à l'AU, ou utiliser une AU à membres dynamiques.

L'appartenance peut être assignée ou dynamique. Une AU dynamique se peuple avec une règle sur les attributs utilisateur ou appareil, exactement comme un groupe dynamique, et consomme les mêmes prérequis de licence.

Les **restricted management administrative units** protègent leurs objets contre les administrateurs portant un rôle à portée tenant. Un utilisateur placé dans une restricted management AU ne peut être modifié que par un administrateur explicitement affecté à cette AU : même un User Administrator au niveau tenant est bloqué. Le Global Administrator, lui, conserve la main. C'est le mécanisme attendu quand un scénario demande d'isoler des comptes sensibles, par exemple des comptes de direction ou des comptes de service, du reste de l'administration.

Côté licence, chaque administrateur affecté à un rôle à portée d'AU consomme une licence Microsoft Entra ID P1. Les AU à membres dynamiques exigent P1 pour chaque membre.

## Utilisateurs

Un compte se crée en cloud-only, se synchronise depuis un Active Directory local, ou arrive par invitation B2B. La propriété `onPremisesSyncEnabled` indique la provenance, et un objet synchronisé se modifie depuis la source, pas depuis Entra.

### Member et Guest

`UserType` vaut `Member` ou `Guest`. C'est une propriété de la **relation à l'organisation**, pas de la provenance de l'identité ni de la façon dont l'utilisateur s'authentifie. Deux conséquences que l'examen exploite :

Un invité qui accepte son invitation ne devient pas Member. L'acceptation fait passer `externalUserState` de `PendingAcceptance` à `Accepted`, et rien d'autre. Le passage à Member est une opération distincte et manuelle.

Les deux notions se croisent. Il existe des **internal guests**, c'est-à-dire des comptes dont les identifiants sont gérés par le tenant mais dont la relation est celle d'un externe, typiquement un prestataire. Et il existe des **external members**, des comptes qui s'authentifient auprès d'un autre tenant mais sont traités comme des membres : c'est exactement ce que produit la cross-tenant synchronization par défaut.

Le statut Guest a des effets concrets. Par défaut, un invité subit des restrictions de lecture de l'annuaire, ne peut pas parcourir la liste complète des utilisateurs et des groupes, et n'apparaît pas dans certaines expériences de partage. Ces restrictions se règlent dans External Identities puis External collaboration settings, avec trois niveaux : accès identique aux membres, accès limité par défaut, ou accès restreint.

### Licences

Une licence s'attribue directement à un utilisateur, ou par appartenance à un groupe (**group-based licensing**). L'affectation par groupe exige Microsoft Entra ID P1 pour le tenant.

Les points testés :

Une licence ne s'applique que si l'utilisateur a un `usageLocation` renseigné. Sans lui, l'affectation échoue, y compris via un groupe, et l'erreur apparaît dans les propriétés de licence de l'utilisateur.

Une licence héritée d'un groupe ne peut pas être retirée individuellement. Il faut sortir l'utilisateur du groupe, ou lui attribuer la licence en direct puis retirer l'héritage.

Les conflits de plans de service se résolvent en désactivant les plans qui se chevauchent, pas en retirant la licence.

En Microsoft Graph, `GET /subscribedSkus` liste les SKU du tenant avec leur `skuId`, leurs `servicePlans` et le nombre d'unités consommées et disponibles. C'est le préalable à toute affectation programmatique, puisqu'il faut le `skuId` pour appeler `POST /users/{id}/assignLicense`.

## Groupes

| | Security group | Microsoft 365 group |
|---|---|---|
| Usage | Autorisation : rôles Azure RBAC, accès applicatif, licences | Collaboration : boîte partagée, SharePoint, Teams |
| Types de membres | Utilisateurs, appareils, service principals, autres groupes | Utilisateurs uniquement |
| Appartenance | Assignée ou dynamique | Assignée ou dynamique |
| Rôle Entra assignable | Oui, si `isAssignableToRole` | Oui, si `isAssignableToRole` |

### Appartenance assignée ou dynamique

Une appartenance assignée se gère à la main. Une appartenance dynamique se calcule à partir d'une règle, soit sur des attributs utilisateur, soit sur des attributs appareil, jamais les deux dans le même groupe.

Les groupes dynamiques exigent **Microsoft Entra ID P1** pour chaque membre du groupe. C'est le discriminant le plus fréquent des questions de ce domaine : un énoncé qui précise que le tenant est en Free élimine mécaniquement toute réponse fondée sur un groupe dynamique.

Une règle porte sur des attributs de l'objet, avec une syntaxe du type `user.department -eq "Finance"` ou `(user.userType -eq "Guest") and (user.companyName -eq "Contoso")`. Le recalcul n'est pas instantané et peut prendre plusieurs minutes après une modification d'attribut.

### Groupes assignables à un rôle

Pour attribuer un rôle Microsoft Entra à un groupe, celui-ci doit avoir été créé avec `isAssignableToRole` à `true`. Trois contraintes en découlent, toutes examinables :

La propriété se fixe **à la création** et ne peut plus être modifiée. Un groupe existant ne peut pas devenir role-assignable.

Un groupe role-assignable ne peut pas avoir d'appartenance dynamique. L'appartenance doit être assignée.

La création exige Privileged Role Administrator ou Global Administrator, et le tenant doit disposer d'une licence P1.

### Propriétaire et membre

Un Owner gère le groupe : il ajoute et retire des membres, modifie les propriétés. Un Member appartient au groupe et bénéficie de ce qui lui est attribué : licences, rôles Azure RBAC, accès applicatifs.

Les deux rôles sont indépendants. Un Owner qui n'est pas Member ne reçoit aucun des accès attribués au groupe. La conséquence à voir dans un scénario de moindre privilège est qu'un Owner peut néanmoins s'auto-ajouter comme membre : il détient donc un chemin d'escalade vers tout ce que le groupe donne. C'est précisément pourquoi les groupes role-assignable font l'objet d'un contrôle de propriété plus strict.

### Cycle de vie des groupes

Deux réglages relèvent de ce domaine. La **naming policy** impose un préfixe ou un suffixe et bloque une liste de mots interdits sur les groupes Microsoft 365. La **expiration policy** fixe une durée de vie, notifie les propriétaires et supprime le groupe sans renouvellement, avec une corbeille de 30 jours. Les deux exigent Microsoft Entra ID P1 pour tous les membres des groupes concernés.

## Identités externes

Quatre mécanismes distincts, souvent confondus.

**External collaboration settings** définit les règles générales de l'invitation B2B : qui a le droit d'inviter (tout le monde, seuls les membres, seuls les administrateurs, personne), quelles restrictions de lecture d'annuaire s'appliquent aux invités, et quels domaines sont autorisés ou bloqués en invitation.

**Cross-tenant access settings** définit la relation avec un tenant Entra partenaire identifié. On y règle l'accès entrant et sortant, application par application si besoin, et surtout les **trust settings** : accepter la MFA effectuée dans le tenant d'origine, accepter le statut de conformité de l'appareil, accepter le statut hybrid joined. Sans ces réglages, un invité soumis à une policy CA exigeant la MFA devra la refaire dans le tenant hôte même s'il vient de la faire chez lui. C'est la réponse attendue à tout scénario du type « nos partenaires se plaignent de faire deux MFA ».

**Cross-tenant synchronization** provisionne automatiquement des utilisateurs d'un tenant vers un autre, sans invitation ni acceptation. Les comptes créés sont par défaut des **external members**, pas des invités, ce qui leur donne les droits de lecture d'annuaire d'un membre. Le scénario typique est un groupe multinational avec plusieurs tenants qui veut une expérience unifiée. La configuration se fait côté tenant source, dans Cross-tenant synchronization, et exige Microsoft Entra ID P1.

**Identity providers** permet aux invités de s'authentifier autrement qu'avec un compte Entra ou un compte Microsoft : fédération directe SAML ou WS-Fed avec l'IdP du partenaire, Google, ou **email one-time passcode**. Ce dernier est le mécanisme de repli activé par défaut depuis octobre 2021 : un invité sans compte Entra ni compte Microsoft reçoit un code à usage unique par courriel.

## Identité hybride

### Choisir le moteur de synchronisation

| | Microsoft Entra Connect Sync | Microsoft Entra Cloud Sync |
|---|---|---|
| Installation | Serveur Windows dédié, moteur complet | Agents légers, plusieurs agents possibles |
| Configuration | Locale, dans l'assistant | Dans le portail Entra |
| Forêts déconnectées | Non | Oui |
| Nombre d'objets par agent | Sans limite pratique | 150 000 objets par périmètre |
| Attributs étendus, filtrage par attribut | Oui | Non |
| Synchronisation des appareils, hybrid join | Oui | Non |
| Pass-through Authentication | Oui | Oui |
| Password writeback | Oui | Oui |
| Device writeback, group writeback avancé | Oui | Limité |
| Exchange hybride complet | Oui | Non |

Le discriminant réel n'est pas « multi-forêts » : les deux gèrent plusieurs forêts. Cloud Sync est le seul à gérer des forêts **déconnectées**, c'est-à-dire sans relation d'approbation entre elles, typiquement après une fusion-acquisition. À l'inverse, Cloud Sync ne sait pas synchroniser les appareils, ce qui exclut le Hybrid Entra join, et ne gère pas les scénarios Exchange hybrides complets.

Les deux peuvent coexister sur le même Active Directory, par exemple Connect Sync sur la forêt historique et Cloud Sync sur une forêt acquise.

### Méthodes d'authentification hybride

**Password Hash Synchronization** synchronise vers Entra un hash du hash NTLM du mot de passe, recalculé toutes les deux minutes. L'authentification se fait alors entièrement dans le cloud. C'est la méthode recommandée par défaut : elle survit à une panne du lien vers le site local, elle alimente la détection de fuites d'identifiants de Microsoft Entra ID Protection, et elle sert de secours si la fédération tombe.

**Pass-through Authentication** valide le mot de passe contre l'Active Directory local au moment de la connexion, via des agents qui n'ouvrent que des connexions sortantes. Le mot de passe n'est jamais stocké dans le cloud. En contrepartie, l'authentification dépend de la disponibilité des agents et du contrôleur de domaine. Microsoft recommande au moins trois agents pour la haute disponibilité. PHS peut être activé en parallèle comme secours.

**Federation** délègue l'authentification à AD FS ou à un fournisseur tiers. C'est l'option la plus lourde, réservée aux exigences que les deux autres ne couvrent pas, comme certaines cartes à puce ou des règles d'authentification tierces.

**Seamless SSO** ajoute une connexion silencieuse pour les utilisateurs déjà authentifiés sur le réseau d'entreprise, via Kerberos. Il s'associe à PHS ou à PTA, jamais à la fédération. Point souvent raté : Seamless SSO exige un appareil **joint au domaine Active Directory**. Il ne s'applique pas aux appareils Microsoft Entra joined ou hybrid joined, qui obtiennent le SSO par leur Primary Refresh Token.

**Password writeback** répercute vers l'Active Directory local les réinitialisations et changements de mot de passe effectués dans le cloud, ce qui rend le self-service password reset utilisable en environnement hybride. Il est pris en charge par Connect Sync comme par Cloud Sync, et exige Microsoft Entra ID P1.

### Diagnostic

Microsoft Entra Connect Health surveille les agents de synchronisation, les serveurs AD FS et les contrôleurs de domaine, et remonte les erreurs de synchronisation. Il exige P1. Les erreurs d'objets, doublons d'attributs et conflits de `userPrincipalName` remontent aussi dans le portail Entra, sous Microsoft Entra Connect puis Sync errors.

## Ce qui se joue sur des détails

Les erreurs les plus fréquentes sur ce domaine ne portent pas sur les définitions mais sur les portées et les licences.

Un rôle Entra à portée d'administrative unit ne donne aucun droit sur les objets hors de l'AU, et placer un groupe dans une AU ne donne pas la main sur ses membres.

Un Owner de groupe ne reçoit rien de ce que le groupe distribue, mais peut se l'octroyer en s'ajoutant comme membre.

Un groupe dynamique, une administrative unit, la cross-tenant synchronization, le group-based licensing, le password writeback et Connect Health exigent tous au minimum Microsoft Entra ID P1. Un énoncé qui mentionne un tenant en édition Free élimine ces réponses.

Un groupe role-assignable ne peut jamais être dynamique, et sa nature se fixe définitivement à la création.

Une licence sans `usageLocation` échoue silencieusement du point de vue de l'utilisateur, et l'erreur ne se voit que dans ses propriétés de licence.
