# Quand utiliser quoi

Table de décision. Chaque ligne donne une réponse orientée examen et, quand c'est utile, le prérequis de licence. Les licences Microsoft Entra évoluent : éviter les raccourcis absolus quand Microsoft distingue désormais capacités historiques P2 et capacités avancées Governance.

Sauf mention contraire, « Free » signifie qu'aucune licence Entra Premium n'est requise pour le mécanisme de base.

## Identités et annuaire

| Besoin | Réponse | Licence / remarque |
|---|---|---|
| Peupler un groupe automatiquement selon un attribut utilisateur | Groupe dynamique, règle sur `user.*` | P1 |
| Peupler un groupe automatiquement selon un attribut appareil | Groupe dynamique de type Device | P1 |
| Attribuer des licences à une population qui évolue | Group-based licensing, avec `usageLocation` renseigné | P1 |
| Attribuer un rôle Microsoft Entra à un groupe | Groupe role-assignable, à appartenance assignée uniquement | P1 |
| Limiter un administrateur à une filiale ou une région | Administrative unit plus rôle Entra à portée d'AU | P1 selon scénario |
| Empêcher les administrateurs à portée tenant de toucher aux comptes sensibles | Restricted management administrative unit | P1 selon scénario |
| Un Global Administrator doit reprendre la main sur un abonnement Azure orphelin | « Access management for Azure resources », qui accorde User Access Administrator à la portée `/` | Free |
| Donner un accès Azure à une équipe | Groupe de sécurité plus role assignment Azure RBAC à la bonne portée | Free |
| Vérifier la propriété d'un custom domain | Enregistrement DNS **TXT** ou **MX** | Free |
| Personnaliser l'écran de connexion | Company Branding | Selon édition / fonctionnalités disponibles |
| Ajouter des métadonnées de sécurité personnalisées | Custom security attributes | Vérifier la licence du scénario |
| Inviter beaucoup d'utilisateurs B2B | Bulk invite CSV / PowerShell | colonnes à reconnaître : `inviteeEmail`, `inviteRedirectUrl` |
| Appareil personnel / BYOD connu par Entra | Microsoft Entra registered | Free |
| Appareil d'entreprise cloud-first | Microsoft Entra joined | Free |
| Appareil joint à AD DS local et présent dans Entra | Hybrid Microsoft Entra joined | Connect Sync pour la synchro appareil |

## Identités externes

| Besoin | Réponse | Licence / remarque |
|---|---|---|
| Faire collaborer un prestataire externe ponctuel | Invitation B2B, `UserType=Guest` | Free |
| Empêcher les utilisateurs d'inviter n'importe qui | External collaboration settings | Free |
| Éviter que les partenaires refassent une MFA déjà faite chez eux | Cross-tenant access settings, trust settings sur la MFA | Free |
| Accepter aussi le statut de conformité d'appareil du partenaire | Cross-tenant access settings, trust settings device compliant / hybrid joined | Free |
| Faire apparaître automatiquement les utilisateurs d'un autre tenant | Cross-tenant synchronization | P1 pour les capacités de base correspondantes |
| Permettre à un externe sans compte Microsoft de se connecter | Email one-time passcode | Free |
| Fédérer l'authentification des invités avec l'IdP du partenaire | Identity provider SAML ou WS-Fed | Free pour le mécanisme de base |
| Nettoyer les accès d'invités à l'expiration d'une assignment | Entitlement Management / access package | capacités historiques en P2, avancées en Governance |

## Identité hybride

| Besoin | Réponse | Licence / remarque |
|---|---|---|
| Synchroniser deux forêts sans relation d'approbation, après une fusion | Cloud Sync | Free |
| Permettre le Hybrid Microsoft Entra join | Connect Sync, seul à synchroniser les appareils | Free |
| Réduire la dépendance à l'infrastructure locale lors de la connexion cloud | Password Hash Synchronization | Free |
| Valider le mot de passe en temps réel contre AD DS sans AD FS | Pass-through Authentication | Free |
| Alimenter la détection d'identifiants divulgués d'ID Protection | Password Hash Synchronization | P2 pour l'exploitation complète du risque |
| Permettre le self-service password reset en hybride | Password writeback | P1 |
| Connexion silencieuse sur des postes joints au domaine Active Directory | Seamless SSO | Free |
| Surveiller la santé des agents de synchronisation et d'AD FS | Microsoft Entra Connect Health | P1 |
| Remplacer AD FS sans dépendance locale forte | PHS, éventuellement préparé en parallèle avant bascule | Free |
| Garder validation AD DS au sign-in sans fédération | PTA + éventuellement Seamless SSO | Free |

## Identités de workload

| Besoin | Réponse | Licence / remarque |
|---|---|---|
| Un workload Azure doit accéder à une ressource sans secret stocké | Managed identity | Free |
| La même identité pour plusieurs ressources Azure | User-assigned managed identity | Free |
| L'identité doit disparaître avec la ressource Azure | System-assigned managed identity | Free |
| Pipeline GitHub Actions / GitLab / Kubernetes hors Azure sans secret | Workload identity federation | Free |
| Démon sans utilisateur connecté | Permission applicative + admin consent + client credentials + `.default` | Free |
| Application au nom de l'utilisateur connecté | Permission déléguée | Free |
| Consentir à un app role Microsoft Graph | Privileged Role Administrator ou Global Administrator | Free |
| Restreindre l'Enterprise Application aux seuls principals affectés | `Assignment required = Yes` | P1 pour certaines affectations par groupe |
| Externaliser les rôles métier d'une application | App roles | Free |
| Service Windows dépendant d'AD DS | Managed service account / gMSA | AD DS |
| Publier une application web interne depuis un navigateur | Application Proxy + préauthentification Entra | P1 |
| Donner accès à une ressource privée en RDP/SSH/TCP/UDP | Microsoft Entra Private Access + private network connector | licence GSA / Entra Suite selon offre |
| Appliquer Conditional Access aux service principals | Conditional Access for workload identities | Workload ID Premium |
| Surveiller le risque d'une workload identity | Workload identity risk | Workload ID Premium |

## Enterprise applications et SaaS

| Besoin | Réponse | Remarque |
|---|---|---|
| Configurer un SSO SaaS SAML | Enterprise Application > Single sign-on > SAML | Entity ID, Reply URL/ACS, claims, certificat |
| Modifier le Name ID ou les claims SAML | Configuration SAML de l'Enterprise Application | pas API permissions |
| Provisionner automatiquement utilisateurs vers SaaS | Provisioning SCIM | analyser avec Provisioning logs |
| Savoir qui a modifié un mapping SCIM | Audit logs | « who changed » = audit |
| Comprendre pourquoi un utilisateur n'a pas été provisionné | Provisioning logs | scope, mapping, statut, étapes |
| Regrouper des applications pour la présentation | Application collections | ne remplace pas affectations / consentement |

## Authentification

| Besoin | Réponse | Licence / remarque |
|---|---|---|
| Amorcer une authentification passwordless pour un nouvel arrivant | Temporary Access Pass | Free |
| N'accepter que des méthodes résistantes au phishing | Authentication strength « Phishing-resistant MFA » | P1 avec CA |
| Imposer la MFA largement sans licence premium | Security Defaults | Free |
| Exiger la MFA pour certains utilisateurs seulement | Conditional Access | P1 |
| Bloquer les mots de passe contenant le nom de l'entreprise, aussi dans AD DS | Microsoft Entra Password Protection + agent DC | P1 |
| Permettre SSPR aux utilisateurs | Self-service password reset | dépend de l'édition / scénario, P1 notamment pour hybride |
| Mesurer l'enregistrement réel des méthodes | User registration details / Registration and reset activity | vérifier l'édition |
| CBA avec binding fort | X509SKI / Subject Key Identifier | CBA |
| Windows Hello for Business hybride vers ressources Kerberos | Cloud Kerberos Trust / Microsoft Entra Kerberos | environnement hybride |
| Révoquer les sessions d'un utilisateur | `Revoke-MgUserSignInSession` | Graph PowerShell |
| Inciter les utilisateurs à enregistrer Authenticator | Registration campaign | ID Protection / Authentication Methods selon fonctionnalité |

## Conditional Access et risque

| Besoin | Réponse | Licence / remarque |
|---|---|---|
| Exiger un appareil conforme pour Exchange Online | Grant control « Require device to be marked as compliant » | P1 + solution de conformité |
| Contrôler les téléchargements pendant une session SaaS | Session policy Defender for Cloud Apps via Conditional Access App Control | licences correspondantes |
| Bloquer entièrement l'accès via Defender for Cloud Apps | Access policy via Conditional Access App Control | licences correspondantes |
| Renforcer uniquement une opération précise | Authentication context | P1 |
| Protéger la modification des policies Conditional Access | Protected actions + authentication context | P1 |
| Compte désactivé, token non expiré, ressource compatible | CAE peut provoquer le rejet anticipé | **les critical events CAE ne se résument pas à “P1 obligatoire”** |
| Rejeter un token hors emplacement autorisé avec évaluation réseau stricte | capacités CAE / Conditional Access correspondantes | P1 selon policy |
| Vérifier qu'une connexion Microsoft 365 transite par le réseau attendu | Compliant network check / Internet Access for Microsoft 365 | GSA / Entra Suite selon offre |
| Mesurer l'effet d'une policy avant activation | Report-only + CA Insights and Reporting | P1 |
| Tester des policies sans vraie connexion | What If | P1 |
| Exiger changement de mot de passe sur user risk | CA / ID Protection user risk | P2 |
| Exiger MFA sur sign-in risk | CA / ID Protection sign-in risk | P2 |

## Defender for Cloud Apps

| Besoin | Réponse |
|---|---|
| Découvrir le Shadow IT | Cloud Discovery |
| Évaluer le risque d'une application SaaS connue | Cloud App Catalog |
| Connecter un service SaaS à Defender for Cloud Apps | Connected app / app connector |
| Gouverner des applications OAuth à risque | OAuth app policies |
| Contrôler la session en temps réel | Conditional Access App Control + session policy |
| Autoriser / bloquer l'entrée via le proxy | Access policy |
| Appliquer des restrictions natives de l'application selon appareil | Application-enforced restrictions |

## Gouvernance

| Besoin | Réponse | Licence / remarque |
|---|---|---|
| Demander un ensemble d'accès avec approbation et expiration | Access package / Entitlement Management | capacités historiques en P2 ; avancées en Governance |
| Déléguer la construction de packages sur un périmètre | Catalog délégué | vérifier la capacité/licence précise |
| Cibler une organisation partenaire | Connected organization | Entitlement Management |
| Exiger l'acceptation d'un PDF avant l'accès | Terms of Use + Conditional Access | ToU dans le périmètre SC-300 |
| Recertifier périodiquement un groupe sensible | Access review | P2 pour capacités historiques |
| Retirer les accès si personne ne répond | Access review + `If reviewers don't respond = Remove access` | P2 |
| Appliquer automatiquement les décisions | `Auto apply results to resource` | Access Review |
| Appliquer manuellement les résultats | laisser Auto apply désactivé puis appliquer après review | Access Review |
| Supprimer les privilèges permanents | PIM avec affectations éligibles | P2 |
| Exiger approbation / ticket à l'élévation | PIM role settings | P2 |
| MFA résistante au phishing au moment de l'élévation | PIM + authentication context CA | P2 + CA |
| Rendre temporaire un accès distribué par un groupe | PIM for Groups | P2 / vérifier capacité précise |
| Gouverner un rôle Azure RBAC temporairement | PIM for Azure resources | P2 |
| Automatiser Joiner/Mover/Leaver | Lifecycle Workflows | Entra ID Governance, complément hors bullet explicite 2026 |
| Garantir l'accès d'urgence | Break-glass cloud-only, GA disponible hors PIM, fortement surveillé | Free pour le principe |

## Supervision et diagnostic

| Besoin | Réponse | Remarque |
|---|---|---|
| Savoir si un utilisateur s'est connecté | Sign-in logs interactive / non-interactive | choisir le bon onglet |
| Application démon authentifiée | Service principal sign-ins | app-only avec SP |
| Azure Function / VM via managed identity | Managed identity sign-ins | onglet distinct |
| Qui a créé/supprimé un secret | Audit logs | champ Initiated by |
| Qui a modifié Conditional Access | Audit logs | configuration |
| Pourquoi un compte n'apparaît pas dans SaaS | Provisioning logs | souvent `Skipped` / scope |
| Pourquoi une connexion est bloquée | Sign-in diagnostic puis onglet Conditional Access | lire code + policy |
| Conserver les logs au-delà de la rétention native | Diagnostic settings vers Log Analytics / Storage / Event Hub | licence + coût de destination |
| Requêter les logs | Log Analytics + KQL | éventuellement Sentinel |
| Évaluer la posture d'identité | Identity Secure Score | ne bloque rien |
