# Quand utiliser quoi

Table de décision. Chaque ligne donne une réponse unique et la licence minimale qui la rend possible, puisqu'un énoncé mentionnant l'édition du tenant élimine à lui seul plusieurs distracteurs.

Sauf mention contraire, « Free » signifie que la fonctionnalité est incluse dans Microsoft Entra ID Free.

## Identités et annuaire

| Besoin | Réponse | Licence |
|---|---|---|
| Peupler un groupe automatiquement selon un attribut utilisateur | Groupe dynamique, règle sur `user.*` | P1 |
| Peupler un groupe automatiquement selon un attribut appareil | Groupe dynamique de type Device | P1 |
| Attribuer des licences à une population qui évolue | Group-based licensing, avec `usageLocation` renseigné | P1 |
| Attribuer un rôle Microsoft Entra à un groupe | Groupe role-assignable, à appartenance assignée uniquement | P1 |
| Limiter un administrateur à une filiale ou une région | Administrative unit plus rôle Entra à portée d'AU | P1 par administrateur |
| Empêcher les administrateurs à portée tenant de toucher aux comptes de direction | Restricted management administrative unit | P1 |
| Un Global Administrator doit reprendre la main sur un abonnement Azure orphelin | Commutateur « Access management for Azure resources », qui accorde User Access Administrator à la portée `/` | Free |
| Donner un accès Azure à une équipe | Groupe de sécurité plus role assignment Azure RBAC à la bonne portée | Free |
| Supprimer automatiquement les groupes Microsoft 365 inutilisés | Group expiration policy | P1 |
| Imposer une convention de nommage aux groupes | Group naming policy | P1 |

## Identités externes

| Besoin | Réponse | Licence |
|---|---|---|
| Faire collaborer un prestataire externe ponctuel | Invitation B2B, `UserType` égal à Guest | Free |
| Empêcher les utilisateurs d'inviter n'importe qui | External collaboration settings, restriction des inviteurs et liste de domaines autorisés | Free |
| Éviter que les partenaires refassent une MFA déjà faite chez eux | Cross-tenant access settings, trust settings sur la MFA | Free |
| Accepter aussi le statut de conformité d'appareil du partenaire | Cross-tenant access settings, trust settings sur device compliant et hybrid joined | Free |
| Faire apparaître automatiquement les utilisateurs d'un autre tenant du groupe | Cross-tenant synchronization, qui crée des external members | P1 |
| Permettre à un externe sans compte Microsoft de se connecter | Email one-time passcode | Free |
| Fédérer l'authentification des invités avec l'IdP du partenaire | Identity provider SAML ou WS-Fed dans External Identities | Free |
| Nettoyer les comptes invités devenus inutiles, sans script | Access package avec expiration, la policy supprimant le compte s'il ne reste aucune assignment | Governance |

## Identité hybride

| Besoin | Réponse | Licence |
|---|---|---|
| Synchroniser deux forêts sans relation d'approbation, après une fusion | Cloud Sync | Free |
| Permettre le Hybrid Microsoft Entra join | Connect Sync, seul à synchroniser les appareils | Free |
| Réduire la dépendance à l'infrastructure locale lors de la connexion cloud | Password Hash Synchronization | Free |
| Ne stocker aucun dérivé de mot de passe dans le cloud | Pass-through Authentication, avec au moins trois agents | Free |
| Alimenter la détection d'identifiants divulgués d'ID Protection | Password Hash Synchronization, condition nécessaire | P2 pour le contenu |
| Permettre le self-service password reset en hybride | Password writeback | P1 |
| Connexion silencieuse sur des postes joints au domaine Active Directory | Seamless SSO | Free |
| Surveiller la santé des agents de synchronisation et d'AD FS | Microsoft Entra Connect Health | P1 |

## Identités de workload

| Besoin | Réponse | Licence |
|---|---|---|
| Un workload sur Azure doit accéder à une ressource sans secret stocké | Managed identity, system-assigned ou user-assigned | Free |
| La même identité pour plusieurs ressources Azure, ou des droits à attribuer avant création | User-assigned managed identity | Free |
| Un pipeline GitHub Actions, GitLab ou Kubernetes hors Azure doit accéder à Entra sans secret | Workload identity federation, federated credential sur l'app registration | Free |
| Un démon doit lire l'annuaire sans utilisateur connecté | Permission applicative plus admin consent, flux client credentials avec scope `.default` | Free |
| Une application agit au nom de l'utilisateur connecté | Permission déléguée | Free |
| Consentir à un app role Microsoft Graph | Privileged Role Administrator ou Global Administrator, Application Administrator ne suffit pas | Free |
| Restreindre l'accès à une enterprise application aux seules personnes affectées | `Assignment required` égal à Yes plus affectations | Free, mais P1 pour affecter des groupes |
| Externaliser les rôles métier d'une application vers Entra | App roles, exposés dans le claim `roles` | Free |
| Publier une application web interne accessible depuis n'importe quel navigateur | Application Proxy, préauthentification Microsoft Entra ID | P1 |
| Donner accès à une ressource interne en RDP, SSH ou tout protocole TCP ou UDP | Microsoft Entra Private Access, avec client Global Secure Access | Entra Suite ou GSA |
| Appliquer du Conditional Access à des service principals | Conditional Access for workload identities | Workload ID Premium |
| Détecter un secret d'application divulgué | Risque sur identités de workload | Workload ID Premium |
| Recertifier les applications détenant des permissions privilégiées | Access review d'identités de workload | Workload ID Premium |

## Authentification

| Besoin | Réponse | Licence |
|---|---|---|
| Amorcer une authentification sans mot de passe pour un nouvel arrivant | Temporary Access Pass, créé par un Authentication Administrator | Free |
| N'accepter que des méthodes résistant au phishing sur une application sensible | Authentication strength « Phishing-resistant MFA » dans un grant control | P1 |
| Imposer la MFA à tout le monde sans licence payante | Security Defaults, en sachant qu'ils excluent le Conditional Access | Free |
| Exiger la MFA pour certains utilisateurs seulement | Policy Conditional Access | P1 |
| Bloquer les mots de passe contenant le nom de l'entreprise, aussi dans l'Active Directory | Microsoft Entra Password Protection, liste personnalisée et agent DC | P1 |
| Permettre aux utilisateurs de réinitialiser leur mot de passe | Self-service password reset | P1 pour les utilisateurs, Free pour les administrateurs |
| Mesurer combien d'utilisateurs ont réellement enregistré une méthode forte | Rapport User registration details, Registration and reset activity | P1 |

## Conditional Access et risque

| Besoin | Réponse | Licence |
|---|---|---|
| Exiger un appareil conforme pour accéder à Exchange Online | Grant control « Require device to be marked as compliant » | P1 plus Intune |
| Contrôler les téléchargements pendant une session SaaS | Session policy Defender for Cloud Apps, avec routage par Conditional Access App Control | P1 plus licence Defender |
| Bloquer entièrement l'accès à une application SaaS non approuvée | Access policy Defender for Cloud Apps, également via Conditional Access App Control | P1 plus licence Defender |
| Renforcer l'authentification uniquement pour une opération précise dans une application | Authentication context, ciblé dans Target resources | P1 |
| Exiger une authentification renforcée pour modifier les policies Conditional Access elles-mêmes | Protected actions liées à un authentication context | P1 |
| Couper la session d'un utilisateur désactivé sans attendre l'expiration du token | Continuous access evaluation, activée par défaut | P1 |
| Rejeter un token présenté depuis une adresse IP non autorisée | Strict location enforcement de la CAE | P1 |
| Vérifier qu'une connexion Microsoft 365 transite bien par le réseau de l'entreprise | Compliant network check, avec Internet Access for Microsoft 365 | Entra Suite ou GSA |
| Mesurer l'effet d'une policy avant de l'activer | Mode report-only, puis classeur Conditional Access Insights and Reporting | P1 |
| Tester le résultat pour un utilisateur donné sans connexion réelle | What If | P1 |
| Exiger un changement de mot de passe quand un compte est probablement compromis | Policy CA sur le risque utilisateur | P2 |
| Exiger une MFA quand une connexion précise est suspecte | Policy CA sur le risque de connexion | P2 |

## Gouvernance

| Besoin | Réponse | Licence |
|---|---|---|
| Permettre de demander un ensemble d'accès cohérent avec approbation et expiration | Access package dans Entitlement Management | Governance |
| Déléguer à un responsable métier la création de packages sur son périmètre | Catalog délégué | Governance |
| Cibler les utilisateurs d'une organisation partenaire dans une policy de demande | Connected organization | Governance |
| Recertifier périodiquement l'appartenance à un groupe sensible | Access review, avec « Auto apply results » activé | P2 |
| Retirer automatiquement les accès sur lesquels personne ne s'est prononcé | Access review, « If reviewers don't respond » égal à Remove access | P2 |
| Recertifier les invités du tenant | Access review ciblant les utilisateurs invités | P2 |
| Supprimer les privilèges permanents des administrateurs | PIM, affectations éligibles au lieu d'actives permanentes | P2 |
| Exiger une approbation et un numéro de ticket à l'élévation | Role settings PIM, approbation et ticket information | P2 |
| Imposer une MFA résistante au phishing au moment précis de l'élévation | Role settings PIM, authentication context Conditional Access à l'activation | P2 |
| Rendre temporaire un accès applicatif distribué par un groupe | PIM for Groups, éligibilité à l'appartenance | P2 |
| Automatiser la création du compte et l'envoi d'un TAP avant l'arrivée | Lifecycle Workflow Joiner, déclenché sur `employeeHireDate` | Governance |
| Désactiver le compte et retirer les accès au départ | Lifecycle Workflow Leaver, déclenché sur `employeeLeaveDateTime` | Governance |
| Garantir l'accès au tenant si une policy verrouille tous les administrateurs | Comptes break-glass, Global Administrator permanent hors PIM, exclus des policies CA, avec alerte sur usage | Free |

## Supervision et diagnostic

| Besoin | Réponse | Licence |
|---|---|---|
| Savoir si un utilisateur s'est connecté | Sign-in logs, onglet interactive ou non-interactive | Free |
| Savoir si une application démon s'est authentifiée | Sign-in logs, onglet Service principal sign-ins | Free |
| Savoir si une Azure Function a pu accéder au Key Vault | Sign-in logs, onglet Managed identity sign-ins | Free |
| Savoir qui a créé ou supprimé un secret d'application | Audit logs, champ Initiated by | Free |
| Savoir qui a modifié une policy Conditional Access | Audit logs | Free |
| Comprendre pourquoi un compte n'apparaît pas dans une application SaaS | Provisioning logs, statut `Skipped` et étape de scope | Free |
| Comprendre pourquoi une connexion a été bloquée | Sign-in diagnostic depuis l'événement, puis onglet Conditional Access | Free |
| Conserver les logs plus de 30 jours | Diagnostic setting vers Log Analytics, storage account ou Event Hub | P1 |
| Requêter les logs et construire des alertes | Log Analytics et KQL, éventuellement Microsoft Sentinel | P1 plus Log Analytics |
| Évaluer la posture d'identité du tenant | Identity Secure Score, sans effet sur les accès | Free |
