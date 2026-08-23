# Synthèse finale

Document de dernière relecture. Il ne remplace aucune fiche : il rassemble les éléments qui se retiennent mal et se vérifient vite.

## Les quatre domaines officiels

| Domaine | Poids | Fiche |
|---|---:|---|
| Implement and manage user identities | 20 à 25 % | [01](01-identites-utilisateurs.md) |
| Implement authentication and access management | 25 à 30 % | [02](02-authentification-acces.md) |
| Plan and implement workload identities | 20 à 25 % | [03](03-identites-workload.md) |
| Plan and automate identity governance | 20 à 25 % | [04](04-gouvernance-identites.md) |

La supervision et le diagnostic ne forment pas un domaine séparé : ils sont répartis dans les quatre.

La fiche [12](12-complements-study-guide-2026.md) rassemble les objectifs explicites du study guide 2026 qui sont moins développés dans les fiches principales.

## Frontières de licence

Les licences Microsoft Entra évoluent. La bonne règle est de distinguer les capacités historiques de P2 des fonctions avancées du produit Microsoft Entra ID Governance.

| Édition | Ce qu'elle débloque notamment |
|---|---|
| **Free** | Utilisateurs, groupes assignés, B2B, Security Defaults, PHS, PTA, Seamless SSO, managed identities, workload identity federation, app roles, logs avec rétention de base |
| **P1** | Conditional Access, groupes dynamiques, group-based licensing, administrative units, groupes role-assignable, SSPR utilisateurs, password writeback, Password Protection, Application Proxy, cross-tenant synchronization, Connect Health, export de logs |
| **P2** | Microsoft Entra ID Protection, PIM, access reviews, et les capacités d'Entitlement Management historiquement incluses dans P2, dont les access packages pour utilisateurs |
| **Entra ID Governance** | Toutes les capacités Governance correspondantes, plus des fonctions avancées comme Lifecycle Workflows et certaines extensions avancées d'Entitlement Management / Access Reviews |
| **Workload ID Premium** | Conditional Access pour service principals, risque sur identités de workload, gouvernance avancée des identités de workload |

Ne pas mémoriser « P2 n'a pas Entitlement Management » : c'est faux. Microsoft indique que les capacités Entitlement Management et Access Reviews historiquement GA dans P2 restent disponibles en P2, tandis qu'Entra ID Governance ajoute des capacités avancées.

## Rôles à citer en moindre privilège

| Opération | Rôle minimal attendu |
|---|---|
| Consentir à un app role Microsoft Graph | Privileged Role Administrator |
| Gérer les app registrations / enterprise applications et consentir aux permissions prises en charge hors app roles Graph | Application Administrator ou Cloud Application Administrator |
| Créer un Temporary Access Pass pour un utilisateur standard | Authentication Administrator |
| Créer un TAP pour un administrateur | Privileged Authentication Administrator |
| Créer un groupe role-assignable | Privileged Role Administrator |
| Lire les journaux sans rien modifier | Reports Reader ou Security Reader |
| Gérer le proxy d'application | Application Administrator, pas Cloud Application Administrator |

## Lire un token

| Observé | Conclusion |
|---|---|
| `scp` présent | Token délégué |
| `idtyp` valant `app` | Token app-only |
| `roles` avec des noms de permissions API, pas de `scp` | Permissions applicatives |
| `roles` avec des noms métier, `scp` également présent | App roles de l'utilisateur dans un token délégué |

Le raccourci « `roles` = application » n'est donc pas absolu. `idtyp=app` est le marqueur fiable d'un token app-only.

## Choisir l'identité d'un workload

Sur Azure, une managed identity : system-assigned si elle doit mourir avec la ressource, user-assigned si elle est partagée ou si les droits doivent précéder la ressource.

Hors d'Azure avec un IdP émettant de l'OIDC, une workload identity federation.

Pour un service Windows dépendant d'AD DS, penser managed service account / gMSA.

En dernier recours seulement : certificat, puis client secret.

## Choisir le journal

| Question | Journal |
|---|---|
| Un utilisateur s'est-il connecté | Sign-in logs, interactive ou non-interactive |
| Une application démon s'est-elle authentifiée | Sign-in logs, Service principal sign-ins |
| Une ressource Azure a-t-elle pu accéder à une autre | Sign-in logs, Managed identity sign-ins |
| Qui a modifié la configuration | Audit logs, champ Initiated by |
| Pourquoi ce compte n'est-il pas dans l'application cible | Provisioning logs, statut / étapes de provisioning |

Trois résultats à interpréter correctement : `Success` dans l'onglet Conditional Access signifie « policy évaluée avec succès » ; `Skipped` dans les Provisioning logs signifie souvent « hors périmètre / rien à faire », pas « en échec » ; « MFA satisfied by claim » signifie qu'aucune MFA n'a été redemandée pour cet événement.

## Micro-détails produit à garder frais

| Sujet | À retenir |
|---|---|
| Vérification custom domain | DNS **TXT ou MX** |
| Bulk B2B CSV | `inviteeEmail`, `inviteRedirectUrl` |
| CBA high-affinity | **X509SKI / SKI** |
| Windows Hello for Business hybride | **Cloud Kerberos Trust** |
| Révoquer les sessions utilisateur | `Revoke-MgUserSignInSession` |
| Client credentials Graph | `https://graph.microsoft.com/.default` |
| Permission déléguée | `scp` |
| Permission applicative app-only | `roles` + `idtyp=app` |
| Application privée TCP/UDP | Entra Private Access + private network connector |
| PDF à accepter avant l'accès | Terms of Use + Conditional Access |
| Shadow IT | Defender for Cloud Apps Cloud Discovery |
| Applications OAuth à risque | OAuth app policies |

## Mots-clés qui décident de la réponse

| Formulation de l'énoncé | Réponse orientée |
|---|---|
| without a signed-in user | Permission applicative, flux client credentials |
| on behalf of the signed-in user | Permission déléguée |
| without storing any secret, workload on Azure | Managed identity |
| without storing any secret, workload outside Azure | Workload identity federation |
| Windows service / AD DS service account | Managed service account / gMSA |
| who changed | Audit logs |
| automatic create, update, disable | Provisioning logs |
| during the session | Session control ou session policy |
| must be assigned to the application | Assignment required |
| eligible, just-in-time, temporary elevation | PIM |
| temporary access to an app delivered by a group | PIM for Groups |
| request, approve, expire | Access package / Entitlement Management |
| periodically confirm, still required | Access review |
| when the employee joins or leaves | Lifecycle Workflow, si le scénario vise cette fonction Governance avancée |
| trust the partner's MFA | Cross-tenant access settings, trust settings |
| users from our other tenant must appear automatically | Cross-tenant synchronization |
| two forests without a trust | Cloud Sync |
| hybrid join required | Connect Sync |
| same identity across several resources | User-assigned managed identity |
| identity must be deleted with the resource | System-assigned managed identity |
| phishing-resistant | Authentication strength |
| only for this specific operation | Authentication context |
| protect the Conditional Access policies themselves | Protected actions |
| account disabled but token still unexpired | CAE peut provoquer le rejet anticipé sur clients / ressources compatibles |
| tenant uses Microsoft Entra ID Free | Éliminer les réponses qui exigent explicitement P1/P2/Governance |
| custom domain ownership | TXT ou MX |
| SAML claim mapping | Enterprise Application > Single sign-on |
| provisioning mapping was changed | Audit logs |
| provisioning execution failed | Provisioning logs |

## Révocation : ne pas confondre les mécanismes

Révoquer un consentement, supprimer un secret ou désactiver un service principal empêche selon le cas l'obtention de **nouveaux** tokens, mais ne fait pas disparaître magiquement tous les access tokens déjà émis.

Pour les utilisateurs, la révocation des sessions invalide les refresh tokens / sessions et CAE peut permettre à des ressources compatibles de rejeter un access token avant son expiration sur certains événements critiques.

## Contrôle mental avant de répondre

1. Quelle identité est en jeu : utilisateur, invité, groupe, service principal, managed identity ?
2. Quelle ressource : une application, Microsoft Graph, une ressource Azure, une application SaaS ?
3. La question porte-t-elle sur l'authentification, l'autorisation, la gouvernance, ou la lecture d'un journal ?
4. L'énoncé mentionne-t-il une édition de licence ? Si oui, éliminer d'abord sur ce critère, mais éviter les vieux raccourcis de licence devenus faux.
5. Demande-t-il explicitement le moindre privilège ? Si oui, choisir le rôle / scope le plus faible qui suffit.
6. Y a-t-il un mot-clé du tableau ci-dessus ? Il tranche souvent entre deux réponses techniquement possibles.
