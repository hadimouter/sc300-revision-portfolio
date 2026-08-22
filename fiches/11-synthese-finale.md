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

## Frontières de licence

C'est le tableau le plus rentable de la révision, parce qu'un énoncé mentionnant l'édition élimine mécaniquement des réponses.

| Édition | Ce qu'elle débloque |
|---|---|
| **Free** | Utilisateurs, groupes assignés, B2B, Security Defaults, PHS, PTA, Seamless SSO, managed identities, workload identity federation, app roles, logs sur 7 jours |
| **P1** | Conditional Access, groupes dynamiques, group-based licensing, administrative units, groupes role-assignable, SSPR utilisateurs, password writeback, Password Protection, Application Proxy, cross-tenant synchronization, Connect Health, export de logs, logs sur 30 jours |
| **P2** | Microsoft Entra ID Protection et policies de risque, PIM y compris PIM for Groups, access reviews |
| **Entra ID Governance** | Entitlement Management, access packages, Lifecycle Workflows, provisioning RH |
| **Workload ID Premium** | Conditional Access pour service principals, risque sur identités de workload, access reviews d'identités de workload |

Le piège classique est de croire que P2 inclut Entitlement Management. Ce n'est pas le cas : Governance est une référence additionnelle.

## Rôles à citer en moindre privilège

| Opération | Rôle minimal |
|---|---|
| Consentir à un app role Microsoft Graph | Privileged Role Administrator |
| Consentir à toute autre permission, toute autre API | Application Administrator ou Cloud Application Administrator |
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

## Choisir l'identité d'un workload

Sur Azure, une managed identity : system-assigned si elle doit mourir avec la ressource, user-assigned si elle est partagée ou si les droits doivent précéder la ressource.

Hors d'Azure avec un IdP émettant du OIDC, une workload identity federation.

En dernier recours seulement, un certificat, puis un client secret.

## Choisir le journal

| Question | Journal |
|---|---|
| Un utilisateur s'est-il connecté | Sign-in logs, interactive ou non-interactive |
| Une application démon s'est-elle authentifiée | Sign-in logs, Service principal sign-ins |
| Une ressource Azure a-t-elle pu accéder à une autre | Sign-in logs, Managed identity sign-ins |
| Qui a modifié la configuration | Audit logs, champ Initiated by |
| Pourquoi ce compte n'est-il pas dans l'application cible | Provisioning logs, statut `Skipped` |

Trois résultats à interpréter correctement : `Success` dans l'onglet Conditional Access signifie « évaluée avec succès » ; `Skipped` dans les Provisioning logs signifie « hors périmètre », pas « en échec » ; « MFA satisfied by claim » signifie qu'aucune MFA n'a été redemandée.

## Mots-clés qui décident de la réponse

| Formulation de l'énoncé | Réponse orientée |
|---|---|
| without a signed-in user | Permission applicative, flux client credentials |
| on behalf of the signed-in user | Permission déléguée |
| without storing any secret, workload on Azure | Managed identity |
| without storing any secret, workload outside Azure | Workload identity federation |
| who changed | Audit logs |
| automatic create, update, disable | Provisioning logs |
| during the session | Session control ou session policy |
| must be assigned to the application | Assignment required |
| eligible, just-in-time, temporary elevation | PIM |
| temporary access to an app delivered by a group | PIM for Groups |
| request, approve, expire | Access package, Entitlement Management |
| periodically confirm, still required | Access review |
| when the employee joins or leaves | Lifecycle Workflow |
| trust the partner's MFA | Cross-tenant access settings, trust settings |
| users from our other tenant must appear automatically | Cross-tenant synchronization |
| two forests without a trust | Cloud Sync |
| hybrid join required | Connect Sync |
| same identity across several resources | User-assigned managed identity |
| identity must be deleted with the resource | System-assigned managed identity |
| phishing-resistant | Authentication strength |
| only for this specific operation | Authentication context |
| protect the Conditional Access policies themselves | Protected actions |
| revoke the session immediately | Continuous access evaluation |
| tenant uses Microsoft Entra ID Free | Éliminer toute réponse exigeant P1 ou plus |

## Contrôle mental avant de répondre

1. Quelle identité est en jeu : utilisateur, invité, groupe, service principal, managed identity ?
2. Quelle ressource : une application, Microsoft Graph, une ressource Azure, une application SaaS ?
3. La question porte-t-elle sur l'authentification, l'autorisation, la gouvernance, ou la lecture d'un journal ?
4. L'énoncé mentionne-t-il une édition de licence ? Si oui, éliminer d'abord sur ce critère.
5. Demande-t-il explicitement le moindre privilège ? Si oui, la bonne réponse est le rôle le plus faible qui suffit, pas Global Administrator.
6. Y a-t-il un mot-clé du tableau ci-dessus ? Il tranche souvent à lui seul entre deux réponses techniquement possibles.
