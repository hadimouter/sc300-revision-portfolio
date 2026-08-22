# Labs

Cinq manipulations réalisées sur un tenant Microsoft Entra de test, documentées pour ce qu'elles démontrent plutôt que pour la suite de clics qu'elles ont demandée.

Chaque lab suit la même trame : ce qu'il cherche à établir, les prérequis de rôle et de licence, ce qui a été fait, ce qui a été observé dans le portail ou dans un token, et ce que cela permet de conclure.

## Environnement

Tenant Microsoft Entra de test en édition Free, avec Security Defaults activés, associé à un abonnement Azure de type pay-as-you-go pour la partie RBAC.

Cette configuration détermine ce qui est démontrable et ce qui ne l'est pas. Les Security Defaults excluent le Conditional Access, et l'édition Free exclut les groupes dynamiques, les administrative units à membres dynamiques, PIM, les access reviews et Microsoft Entra ID Protection. Les labs portent donc sur ce que le socle permet réellement de vérifier : la séparation identité et autorisation, le modèle d'application, le flux app-only, et la lecture des journaux.

Les notions non couvertes par ces labs sont traitées uniquement sur le plan théorique dans les fiches, et le [README](../README.md) le signale explicitement.

## Parcours

| Lab | Sujet | Ce qu'il établit |
|---|---|---|
| [01](01-tenant-users-groups-rbac.md) | Tenant, utilisateurs, groupes, Azure RBAC | Un groupe est un vecteur d'autorisation ; un Owner n'est pas un Member |
| [02](02-b2b-administrative-units.md) | B2B et administrative units | Une identité externe n'est pas un accès ; une AU n'est pas transitive |
| [03](03-app-registration-graph.md) | App registration, service principal, Microsoft Graph | Deux objets, un Client ID ; le contenu d'un token app-only |
| [04](04-authentification-logs-tap.md) | Authentification, journaux, Temporary Access Pass | Où se lit quoi, et ce que signifie réellement une MFA satisfaite par claim |
| [05](05-moindre-privilege.md) | Retrait d'accès et moindre privilège | Révoquer un consentement ne coupe pas une session en cours |

## Captures d'écran

Chaque lab est illustré par les écrans qui établissent son point, rassemblés dans [screenshots](../screenshots/).

Toutes ont été relues avant publication. Les valeurs sensibles sont couvertes par des aplats opaques, jamais par un flou : Tenant ID, Object ID, noms d'utilisateur principaux, adresse IP, identifiants de corrélation et de session, valeur du client secret, identifiant du Temporary Access Pass. Les comptes qui apparaissent (Alice Marketing, Bob Security, Charlie External) sont des comptes de test créés pour ces manipulations.

Le tri a été volontairement sévère : une capture n'est publiée que si elle démontre quelque chose que le texte se contente d'affirmer. Les écrans de navigation et les étapes intermédiaires ne le sont pas.
