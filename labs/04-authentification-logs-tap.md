# Lab 4. Authentification, journaux et Temporary Access Pass

## Ce que le lab établit

Que chaque type d'identité produit sa propre trace, que les trois journaux répondent à trois questions différentes, et qu'une MFA satisfaite par un claim n'est pas une MFA rejouée.

## Prérequis

| Élément | Valeur |
|---|---|
| Rôle pour créer un TAP | Authentication Administrator pour un compte standard, Privileged Authentication Administrator pour un compte administrateur |
| Rôle pour lire les journaux | Reports Reader suffit |
| Licence | Aucune. Le TAP et la lecture des journaux sur 7 jours sont disponibles en édition Free |

Le tenant fonctionnant sous Security Defaults, aucune policy Conditional Access n'existe et l'onglet correspondant des journaux reste vide. C'est une information en soi, et non une absence de données : sous Security Defaults, la MFA est imposée globalement, sans ciblage ni condition, et l'onglet Conditional Access d'un log ne peut donc rien afficher.

## Manipulations, journaux de connexion utilisateur

Connexion réelle avec un compte utilisateur, puis lecture de l'événement dans Sign-in logs.

Examen de l'onglet Authentication Details, de l'état de l'appareil et de l'emplacement.

## Observations

L'onglet Authentication Details expose une ligne par étape d'authentification, avec la méthode, le détail de la méthode, le résultat, et la policy d'authentification qui l'a exigée.

Sur une connexion consécutive à une première authentification déjà réalisée, le résultat affiché est **« MFA requirement satisfied by claim in the token »**. Cela signifie que l'exigence a été satisfaite par un claim MFA déjà présent dans le token, issu de l'authentification précédente. Aucune nouvelle interaction n'a été demandée à l'utilisateur, et aucune ne figure dans le journal.

C'est le comportement normal du single sign-on, et c'est ce qui explique le signalement récurrent « la policy exige la MFA mais on ne me la demande jamais ». La réponse n'est pas de chercher une défaillance de la policy mais de régler la sign-in frequency, ce qui suppose du Conditional Access, donc P1.

## Manipulations, connexions de service principal

Déclenchement du client credentials flow du [lab 3](03-app-registration-graph.md), puis recherche de la trace correspondante.

## Observations

L'événement n'apparaît dans aucun des deux onglets de connexions utilisateur. Il figure dans l'onglet **Service principal sign-ins**, sans utilisateur associé, avec la ressource Microsoft Graph et un type de credential valant Client Secret.

Le journal comporte quatre onglets distincts : User sign-ins interactive, User sign-ins non-interactive, Service principal sign-ins et Managed identity sign-ins. Le quatrième est réservé aux managed identities et n'aurait pas reçu cet événement, l'application s'authentifiant ici avec un secret et non avec une identité gérée par Azure. Envoyer par réflexe toute authentification de workload vers l'onglet des service principals est une erreur : une Azure Function accédant à un Key Vault se trace dans l'onglet des managed identities.

## Manipulations, journaux d'audit

Lecture des Audit logs sur la période couvrant le lab 3.

## Observations

Chaque opération de configuration apparaît avec son acteur dans le champ Initiated by :

- le consentement délégué accordé sur `User.Read`
- l'app role assignment de `User.Read.All` sur le service principal
- l'ajout du client secret
- la suppression du client secret

Aucune de ces entrées ne figure dans les Sign-in logs, et aucune authentification ne figure dans les Audit logs. La séparation est nette : le premier journal répond à « qui s'est authentifié », le second à « qui a modifié la configuration ».

## Manipulations, Temporary Access Pass

Activation de la méthode Temporary Access Pass dans Protection puis Authentication methods, avec un ciblage restreint à un groupe.

Génération d'un TAP pour un utilisateur de test, depuis la fiche de cet utilisateur, onglet Authentication methods.

Connexion réelle avec ce TAP, puis lecture de l'événement.

## Observations

La policy expose la durée de vie minimale et maximale, le délai avant activation, et un réglage **One-time use**. Ce dernier ne rend pas les pass à usage unique par lui-même : tant qu'il vaut False, l'administrateur choisit à chaque création si le pass est à usage unique ou multiple. C'est en le passant à True que l'usage unique devient obligatoire pour tous les pass.

Le pass ne peut pas être généré pour soi-même. L'écran n'est pas disponible sur son propre compte, ce qui est cohérent avec le rôle Authentication Administrator, prévu pour agir sur les comptes d'autrui.

À la connexion, l'onglet Authentication Details affiche explicitement **Temporary Access Pass** comme méthode, et l'exigence de MFA est marquée satisfaite. Le TAP a donc bien valeur de MFA, ce qui est précisément ce qui permet à son porteur d'enregistrer une méthode dans un tenant exigeant la MFA pour l'enregistrement.

Activer la méthode dans la policy n'a créé aucun pass. Autoriser une méthode et la provisionner pour un utilisateur donné sont deux opérations distinctes, ce qui vaut d'ailleurs pour toutes les méthodes.

## Ce qu'il faut en retenir

Le choix du journal se déduit du type de question, pas du type d'incident. « Qui s'est authentifié » va aux Sign-in logs, « qui a changé quoi » aux Audit logs, et le type d'identité détermine l'onglet.

Une MFA satisfaite par claim est une MFA non redemandée, et c'est normal.

Une méthode autorisée n'est pas une méthode enregistrée, et l'écart entre les deux se mesure dans le rapport User registration details.

## Lien avec l'examen

Domaines « Implement authentication and access management » et « Monitor identity activity ». Voir les fiches [02](../fiches/02-authentification-acces.md) et [05](../fiches/05-supervision-diagnostic.md).
