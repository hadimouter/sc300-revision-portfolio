# Simulation miroir SC-300 — 50 questions originales

> **Important** : cette simulation ne reproduit aucune question Microsoft Learn ou d'examen. Les scénarios, noms, formulations et distracteurs sont originaux. Elle reprend uniquement des compétences publiques du guide SC-300 et des types de raisonnements rencontrés pendant la préparation.

Objectif : vérifier le **transfert de connaissances**, pas la reconnaissance d'une banque de questions.

## Questions

### 1
Northwind possède trois divisions. Pour chaque division, une unité administrative a été créée. Le groupe de sécurité `GRP-Finance` a été ajouté à l’unité administrative `AU-Finance`, et Alice est membre de `GRP-Finance`.

Un administrateur du support reçoit le rôle **Helpdesk Administrator** limité à `AU-Finance`, mais il ne peut pas réinitialiser le mot de passe d’Alice.

Que devez-vous faire en respectant le moindre privilège ?

A. Ajouter Alice directement à `AU-Finance`  
B. Ajouter le rôle Global Administrator au support  
C. Convertir `GRP-Finance` en groupe dynamique  
D. Affecter une licence P2 à Alice

### 2
Une application daemon utilise la permission **Application** `Mail.Read` sur Microsoft Graph et le flux client credentials.

Quel scope doit être envoyé au point de terminaison OAuth 2.0 ?

A. `Mail.Read`  
B. `https://graph.microsoft.com/Mail.Read`  
C. `https://graph.microsoft.com/.default`  
D. `openid profile offline_access`

### 3
Votre équipe utilise Microsoft Defender for Cloud Apps pour évaluer le niveau de risque des applications cloud.

Quelles sont les catégories générales utilisées dans le score des applications ?

A. Général, sécurité, conformité et juridique  
B. Identité, réseau, appareil et authentification  
C. Confidentialité, utilisateur, coût et géolocalisation  
D. Connecteur, stratégie, conformité et gouvernance

### 4
Une entreprise veut empêcher l’accès aux applications Microsoft 365 depuis des appareils marqués **non conformes**.

Quelle fonctionnalité devez-vous configurer ?

A. Une stratégie Conditional Access  
B. Une campagne d’enregistrement MFA  
C. Microsoft Entra Application Proxy  
D. Une stratégie de mot de passe

### 5
Deux machines virtuelles Azure exécutent la même application. Elles doivent utiliser **exactement la même identité** pour accéder à un compte de stockage afin de réduire le nombre d’attributions RBAC à administrer.

Quel type d’identité devez-vous utiliser ?

A. Une identité managée affectée par le système  
B. Une identité managée affectée par l’utilisateur  
C. Un compte utilisateur Microsoft Entra  
D. Un compte gMSA

### 6
Vous devez retrouver toutes les révisions d’accès pour lesquelles une décision **Deny** a été appliquée à des membres d’un groupe Microsoft 365.

Où devez-vous rechercher ces informations ?

A. Journaux d’audit Microsoft Entra  
B. Journaux d’activité Azure  
C. Microsoft Purview eDiscovery  
D. Journaux de connexion Microsoft Entra

### 7
Vous ajoutez `adatum.net` comme domaine personnalisé dans Microsoft Entra.

Quels enregistrements DNS peuvent être utilisés pour vérifier la propriété du domaine ? **Choisissez deux réponses.**

A. MX  
B. TXT  
C. CNAME  
D. SRV  
E. AAAA

### 8
Un ingénieur doit pouvoir devenir **Security Administrator** uniquement lorsqu’il en a besoin. Chaque activation doit être approuvée par un responsable.

Comment devez-vous configurer son rôle ?

A. Affectation active permanente  
B. Affectation éligible  
C. Affectation directe par Azure RBAC  
D. Groupe dynamique de sécurité

### 9
Une application utilise le flux client credentials. Votre politique sécurité interdit les secrets partagés.

Quelle combinaison convient le mieux pour authentifier l’application ?

A. Client ID + mot de passe utilisateur  
B. Client ID + certificat X.509  
C. Client ID + access token statique  
D. Mot de passe + refresh token

### 10
Vous préparez un fichier CSV pour inviter en masse des partenaires externes dans Microsoft Entra.

Que doit contenir la **première ligne** du fichier ?

A. Les en-têtes de colonnes  
B. Les valeurs du premier utilisateur  
C. Le schéma URI Microsoft Graph  
D. Le numéro de version du modèle CSV

### 11
Une application interne est publiée via Microsoft Entra Application Proxy sous l’URL externe `https://portal.contoso.com`.

Vous devez faire pointer ce nom vers le domaine fourni par Microsoft pour Application Proxy.

Quel type d’enregistrement DNS faut-il créer ?

A. TXT  
B. SRV  
C. CNAME  
D. AAAA

### 12
Un utilisateur affirme que son authentification MFA a été refusée.

Quel journal devez-vous examiner en premier pour comprendre ce qui s’est produit pendant cette tentative ?

A. Audit logs  
B. Provisioning logs  
C. Sign-in logs  
D. Azure Activity logs

### 13
Vous devez créer des groupes de sécurité dont l’appartenance sera calculée automatiquement à partir d’attributs.

Quels types d’appartenance sont utilisables pour ce besoin ? **Choisissez deux réponses.**

A. Dynamic User  
B. Dynamic Device  
C. Assigned  
D. Distribution List  
E. Mail-enabled Security

### 14
`AU-Europe` contient le groupe `GRP-Europe`, mais pas directement les utilisateurs qui sont membres de ce groupe.

Bob possède le rôle **User Administrator** limité à `AU-Europe`.

Quelle opération peut-il effectuer grâce à la présence de `GRP-Europe` dans l’AU ?

A. Réinitialiser le mot de passe de tous les membres du groupe  
B. Gérer l’appartenance du groupe lui-même  
C. Attribuer Global Administrator aux membres  
D. Modifier tous les appareils des membres du groupe

### 15
Un service SaaS crée automatiquement des utilisateurs dans votre tenant Microsoft Entra. Les données de diagnostic sont envoyées dans Log Analytics.

Quelle table KQL est la plus adaptée pour suivre ces créations via le moteur de provisioning ?

A. `AADProvisioningLogs`  
B. `AADRiskyUsers`  
C. `SigninLogs`  
D. `AuditLogsAzureResources`

### 16
Vous préparez une nouvelle stratégie Conditional Access et vous voulez mesurer ses effets sans bloquer un seul utilisateur.

Que devez-vous faire ?

A. Activer la stratégie pour 5 % des utilisateurs  
B. Placer la stratégie en mode Report-only  
C. Désactiver toutes les autres stratégies  
D. Configurer uniquement une exclusion Global Administrator

### 17
Vous importez manuellement dans Defender for Cloud Apps un journal provenant d’un pare-feu non directement reconnu.

Vous devez indiquer à l’analyseur la nature générique du journal.

Quelle source devez-vous sélectionner ?

A. Microsoft 365  
B. Azure Firewall  
C. Other  
D. Defender for Endpoint

### 18
Vous voulez empêcher les utilisateurs non administrateurs d’ouvrir le centre d’administration Microsoft Entra.

Dans quelle catégorie de paramètres devez-vous intervenir ?

A. Device settings  
B. Group settings  
C. User settings  
D. Protected actions

### 19
Un consultant externe appartenant au tenant Fabrikam doit pouvoir accéder à votre tenant et **énumérer tous les utilisateurs et groupes**, sans disposer de droits d’administration.

Quelles actions devez-vous effectuer ? **Choisissez deux réponses.**

A. L’inviter comme Guest  
B. Lui attribuer Directory Readers  
C. Lui attribuer Global Administrator  
D. Créer un compte cloud-only Member  
E. Lui attribuer User Administrator

### 20
Dans Defender for Cloud Apps, un utilisateur doit pouvoir gérer les alertes, créer et modifier des file policies, autoriser les actions de gouvernance de fichiers et consulter les rapports de gestion des données.

Quel rôle suit le mieux le principe du moindre privilège ?

A. Security Administrator  
B. Compliance Administrator  
C. Global Reader  
D. Authentication Administrator

### 21
Une VM Azure utilise une identité managée affectée par le système. Cette identité possède déjà le rôle **Storage File Data SMB Share Contributor** au niveau du compte de stockage `storageA`.

Vous créez un autre partage dans `storageB`. La VM doit pouvoir monter ce nouveau partage en appliquant le moindre privilège.

Que devez-vous faire ?

A. Attribuer le même rôle à l’identité au niveau de `storageB`  
B. Attribuer Reader sur l’abonnement  
C. Attribuer Owner sur le Resource Group  
D. Déplacer l’attribution actuelle au niveau de l’abonnement

### 22
Des utilisateurs invités invitent actuellement d’autres personnes dans votre tenant.

Vous voulez que seuls certains administrateurs et des utilisateurs approuvés puissent envoyer des invitations B2B.

Que devez-vous configurer ?

A. Cross-tenant access settings  
B. External collaboration settings  
C. Group settings  
D. Conditional Access

### 23
Votre environnement hybride utilise Microsoft Entra Connect Sync.

Lorsqu’un compte est désactivé dans l’Active Directory local, vous voulez que la prochaine tentative d’authentification cloud échoue immédiatement sans attendre une synchronisation de hash.

Quelle méthode convient le mieux ?

A. Password Hash Synchronization  
B. Pass-through Authentication  
C. Password Writeback  
D. Federation uniquement

### 24
Votre organisation possède de nombreux abonnements Azure regroupés en plusieurs management groups.

Vous voulez organiser les Access Reviews selon cette hiérarchie afin d’en faciliter la gestion.

Quel composant de gouvernance utilisez-vous ?

A. Catalogs  
B. Programs  
C. Access Packages  
D. Connected Organizations

### 25
Vous venez d’activer Cloud Discovery dans Defender for Cloud Apps, mais aucun rapport n’est disponible.

Vous disposez d’un fichier de logs réseau exporté d’un équipement compatible et vous voulez obtenir immédiatement une première analyse.

Que devez-vous créer ?

A. Une access policy  
B. Un snapshot report  
C. Un application connector  
D. Une session policy

### 26
Vous utilisez Pass-through Authentication et Smart Lockout.

Quelles configurations permettent à Microsoft Entra de bloquer une attaque avant que l’Active Directory local ne verrouille le compte ? **Choisissez deux réponses.**

A. Le seuil Entra doit être inférieur au seuil AD DS  
B. Le seuil Entra doit être supérieur au seuil AD DS  
C. La durée de verrouillage Entra doit être supérieure à celle d’AD DS  
D. La durée Entra doit toujours être zéro  
E. Un RODC est obligatoire

### 27
Votre entreprise souhaite déployer Windows Hello for Business dans un environnement hybride tout en évitant une infrastructure PKI complexe.

Quelle option d’authentification devez-vous privilégier ?

A. Certificate Trust  
B. Key Trust  
C. Cloud Kerberos Trust  
D. Password Hash Synchronization uniquement

### 28
Vous utilisez App Governance dans Defender for Cloud Apps.

Vous voulez repérer les applications qui possèdent beaucoup plus de permissions Microsoft Graph qu’elles n’en utilisent réellement.

Quel modèle de stratégie est le plus adapté ?

A. Nouvelle application non certifiée  
B. Application avec privilèges excessifs  
C. Nouvelle application à faible privilège  
D. Application inactive

### 29
Une application d’entreprise doit mettre en œuvre un SSO basé sur un protocole fédéré.

Quelles méthodes sont adaptées à ce type de scénario ? **Choisissez trois réponses.**

A. OAuth  
B. OpenID Connect  
C. SAML  
D. Password-based SSO  
E. Linked SSO

### 30
Votre entreprise veut utiliser à la fois Microsoft Entra Private Access et Microsoft Entra Internet Access.

Vous devez choisir l’offre qui couvre les deux en limitant les coûts et la multiplication des licences.

Quelle offre convient ?

A. Microsoft Entra ID P1  
B. Microsoft Entra ID P2  
C. Microsoft Entra Suite  
D. Microsoft Entra Workload ID

### 31
Une Access Review récurrente utilise les managers comme reviewers.

Certains managers ne répondent jamais et vous voulez éviter que la revue reste sans décision.

Quelles actions devez-vous configurer ? **Choisissez deux réponses.**

A. Créer une revue récurrente  
B. Configurer des fallback reviewers  
C. Désactiver l’auto-apply  
D. Convertir la revue en Access Package  
E. Supprimer les reviewers principaux

### 32
Vous configurez le Self-Service Password Reset pour des comptes administrateurs Microsoft Entra.

Quelle méthode ne peut pas être utilisée par ces comptes pour vérifier leur identité ?

A. Mobile app notification  
B. Mobile phone  
C. Email  
D. Security questions

### 33
Un analyste doit uniquement consulter les rapports et journaux Microsoft Entra, sans modifier la configuration.

Quel rôle lui attribuez-vous ?

A. Reports Reader  
B. Security Administrator  
C. Privileged Role Administrator  
D. Authentication Administrator

### 34
Vous exportez des résultats de journaux d’audit depuis Microsoft Entra.

Quels formats sont adaptés à l’export ? **Choisissez deux réponses.**

A. CSV  
B. JSON  
C. Windows Event Log  
D. TXT uniquement

### 35
Une application privée hébergée sur un serveur Windows local doit être accessible via Microsoft Entra Private Access.

Quel composant doit être installé dans le réseau privé ?

A. Global Secure Access Client  
B. Private Network Connector  
C. Entra Provisioning Agent  
D. Azure Arc Agent

### 36
Vous enquêtez sur une connexion datant de plusieurs semaines et vous voulez identifier **l’événement exact ayant déclenché le risque**.

Quel rapport devez-vous consulter ?

A. Risky users  
B. Risk detections  
C. Risky sign-ins  
D. Audit logs

### 37
Tous les employés du département `Research` doivent automatiquement recevoir une licence Microsoft 365 E5.

L’attribut `department` est renseigné correctement pour tous les utilisateurs.

Quelle solution nécessite le moins d’administration ?

A. Un Access Package  
B. Un groupe dynamique d’utilisateurs avec group-based licensing  
C. Une Lifecycle Workflow  
D. Une attribution individuelle par PowerShell

### 38
Votre entreprise veut conserver l’application des stratégies de sécurité utilisateur AD DS tout en réduisant la complexité par rapport à AD FS.

Quelle combinaison d’authentification hybride convient le mieux ?

A. Federation + Seamless SSO  
B. Pass-through Authentication + Seamless SSO  
C. Password Hash Synchronization uniquement  
D. Password Writeback + Federation

### 39
Vous utilisez Entitlement Management.

Vous devez ajouter un groupe existant comme ressource à un catalogue au moyen de Microsoft Graph PowerShell.

Quelle commande est la plus appropriée ?

A. `New-MgEntitlementManagementAccessPackage`  
B. `New-MgEntitlementManagementAccessPackageResourceRequestCatalog`  
C. `New-MgEntitlementManagementAccessPackageResourceRoleScope`  
D. `New-MgGroup`

### 40
Une entreprise veut régulièrement réexaminer les accès de ses utilisateurs et **appliquer automatiquement les décisions prises à la fin de chaque revue**.

Quelle solution répond le mieux à ce besoin ?

A. Conditional Access  
B. Access Reviews avec application automatique des résultats  
C. Privileged Identity Management  
D. Group expiration policy

### 41
Tous les employés utilisent des postes Windows 11 Microsoft Entra joined.

Vous devez proposer une méthode MFA forte qui ne nécessite pas l’utilisation d’un téléphone mobile.

Quelle méthode est la plus adaptée ?

A. Windows Hello for Business  
B. SMS  
C. Microsoft Authenticator push  
D. Voice call

### 42
Vous concevez une stratégie Conditional Access comportant un contrôle de session `Sign-in frequency`.

Vous voulez estimer le résultat de la stratégie pour un utilisateur et une application donnés.

Quel outil utilisez-vous ?

A. Azure CLI  
B. Microsoft Graph Explorer  
C. What If / outil d’analyse de scénario  
D. Audit logs

### 43
Vous implémentez une policy basée sur le **user risk**.

Vous voulez limiter les interruptions pour les utilisateurs et ne cibler que les situations les plus graves.

Quel niveau de risque choisissez-vous ?

A. Low  
B. Medium  
C. High  
D. Aucun niveau

### 44
Vous avez plusieurs partenaires B2B, mais seul le tenant de `Litware` doit pouvoir accéder à vos ressources.

Quelle configuration correspond le mieux ?

A. Autoriser tous les accès entrants par défaut  
B. Bloquer globalement les accès et créer une exception spécifique pour Litware  
C. Autoriser tous les tenants mais exiger MFA  
D. Créer un tenant Microsoft Entra séparé pour Litware

### 45
Vous ajoutez une entreprise partenaire comme **Connected Organization** dans Entitlement Management.

Ses utilisateurs ne possèdent pas nécessairement de compte Microsoft Entra.

Quelles méthodes peuvent permettre leur authentification ? **Choisissez deux réponses.**

A. Federation  
B. Email one-time passcode  
C. Client credentials flow  
D. Device code flow uniquement

### 46
Vous devez gérer automatiquement un parc de 2 000 appareils Windows Microsoft Entra joined.

Le groupe doit se remplir en fonction des propriétés des appareils.

Quel type de groupe créez-vous ?

A. Microsoft 365 group avec Dynamic User  
B. Security group avec Dynamic Device  
C. Security group Assigned  
D. Distribution group

### 47
Parmi les méthodes suivantes, laquelle constitue le meilleur choix principal pour une MFA résistante au phishing ?

A. FIDO2 security key  
B. Software OATH  
C. Hardware OATH  
D. Voice call

### 48
Vous créez une Access Review concernant l’affectation d’un utilisateur à un **rôle Microsoft Entra personnalisé**.

Vous devez donner au reviewer le rôle le moins privilégié permettant de gérer correctement ce type de revue et ses notifications.

Quel rôle choisissez-vous ?

A. User Administrator  
B. Privileged Role Administrator  
C. Security Administrator  
D. Global Reader

### 49
Une équipe développe une nouvelle application destinée à accepter des identités provenant de fournisseurs sociaux.

Quel objet devez-vous créer dans Microsoft Entra pour représenter cette application et configurer son authentification ?

A. Une App Registration  
B. Une Intune app configuration policy  
C. Une Azure Policy  
D. Une Application Proxy connector group

### 50
Votre organisation utilise PIM for Groups.

Vous voulez examiner les événements liés aux activations PIM et aux modifications d’appartenance d’un groupe privilégié.

Quelle source devez-vous utiliser ?

A. Microsoft Purview Unified Audit Log  
B. Microsoft Entra resource audit  
C. Microsoft Entra Access Reviews  
D. Azure Subscription Activity Log

## Corrigé

<details>
<summary>Afficher les réponses</summary>

1. A  
2. C  
3. A  
4. A  
5. B  
6. A  
7. A, B  
8. B  
9. B  
10. D  
11. C  
12. C  
13. A, B  
14. B  
15. A  
16. B  
17. C  
18. C  
19. A, B  
20. B  
21. A  
22. B  
23. B  
24. B  
25. B  
26. A, C  
27. C  
28. B  
29. A, B, C  
30. C  
31. A, B  
32. D  
33. A  
34. A, B  
35. B  
36. B  
37. B  
38. B  
39. B  
40. B  
41. A  
42. C  
43. C  
44. B  
45. A, B  
46. B  
47. A  
48. B  
49. A  
50. B

</details>

## Utilisation conseillée

- Répondre une première fois sans ouvrir le corrigé.
- Noter séparément les erreurs de **concept**, de **micro-détail Microsoft** et de **lecture du scénario**.
- Refaire uniquement les compétences ratées avec de nouveaux scénarios, plutôt que mémoriser la formulation de cette simulation.
