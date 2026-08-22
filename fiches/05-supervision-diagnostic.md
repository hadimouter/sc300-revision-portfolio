# Supervision et diagnostic

Ce domaine n'existe pas comme bloc autonome dans les compétences mesurées. Il est réparti sous « Monitor identity activity by using logs, workbooks, and reports », rattaché à l'analyse et à la résolution d'incidents, et il alimente des questions dans les quatre domaines officiels.

Sur le terrain comme à l'examen, tout se ramène à choisir le bon journal. Trois questions, trois journaux, et se tromper de journal fait perdre le point même quand le raisonnement est juste.

| Question | Journal |
|---|---|
| Qui s'est authentifié, et cela a-t-il réussi | Sign-in logs |
| Qui a modifié la configuration de l'annuaire | Audit logs |
| Qu'a tenté le moteur de provisioning vers une application cible | Provisioning logs |

## Sign-in logs

Le journal de connexion comporte exactement **quatre catégories**, présentées en quatre onglets. Les confondre est une erreur classique.

**User sign-ins (interactive)** enregistre les connexions où l'utilisateur a fourni un facteur d'authentification : mot de passe, réponse à une notification, clé de sécurité.

**User sign-ins (non-interactive)** enregistre les connexions effectuées pour le compte d'un utilisateur par un client, sans interaction : un client de messagerie qui rafraîchit son token, une application qui utilise un refresh token. Le volume y est bien supérieur, et c'est là qu'on trouve les authentifications héritées.

**Service principal sign-ins** enregistre les authentifications d'applications utilisant leur propre identité, avec un secret ou un certificat. Il n'y a jamais d'utilisateur associé.

**Managed identity sign-ins** enregistre les authentifications de managed identities. C'est une catégorie **distincte** des service principal sign-ins, et le piège est réel : une question du type « le job Azure Function n'accède plus au Key Vault, où regardez-vous » attend cet onglet, pas celui des service principals.

### Lire une entrée

Les champs à examiner, dans l'ordre où ils servent : l'identité et l'application, la ressource visée, le statut avec son code d'erreur, l'onglet **Authentication Details** qui détaille chaque étape d'authentification, l'onglet **Conditional Access** qui liste les policies évaluées, l'appareil et son état de conformité, l'emplacement et l'adresse IP, et enfin les identifiants de diagnostic.

Le **Request ID** identifie une requête unique. Le **Correlation ID** regroupe plusieurs requêtes appartenant à la même opération, par exemple les redirections successives d'un flux d'authentification. Le premier sert à pointer un événement précis dans un ticket de support Microsoft, le second à reconstituer une séquence complète.

### Authentication Details

Cet onglet donne, étape par étape, la méthode utilisée, le détail de la méthode, le résultat, et la policy d'authentification qui l'a exigée.

Le résultat **« MFA requirement satisfied by claim in the token »** signifie que l'exigence a été satisfaite par un claim MFA déjà présent dans le token, issu d'une authentification antérieure. Aucune nouvelle MFA n'a été demandée à l'utilisateur pour cet événement, et aucune n'a été journalisée. C'est le comportement normal du single sign-on, pas une anomalie, et un scénario du type « l'utilisateur dit qu'on ne lui a pas demandé de MFA alors que la policy l'exige » se résout par cette lecture, éventuellement complétée par un réglage de sign-in frequency.

### Conditional Access dans les logs

L'onglet liste chaque policy évaluée avec son résultat, et la sémantique est contre-intuitive :

| Résultat | Signification |
|---|---|
| Success | La policy s'appliquait et ses contrôles ont été satisfaits |
| Failure | La policy s'appliquait et ses contrôles n'ont pas été satisfaits, l'accès a été refusé |
| Not applied | La policy existe mais ses conditions d'affectation n'étaient pas réunies pour cette connexion |
| Not enabled | La policy est désactivée, ou en report-only pour la colonne correspondante |

`Success` ne veut donc pas dire « la policy a accordé l'accès » mais « la policy a été évaluée avec succès ». Une connexion bloquée présente au moins une policy en `Failure`. Une connexion qu'on croyait protégée et qui affiche `Not applied` partout signale un problème de ciblage, pas de contrôle.

## Audit logs

Les Audit logs tracent les modifications de configuration de l'annuaire : ajout ou retrait d'un membre de groupe, création ou suppression d'utilisateur, consentement à une application, app role assignment, création et suppression d'un secret ou d'un certificat, modification d'une policy Conditional Access, activation d'un rôle PIM, modification d'une configuration de provisioning.

Chaque entrée porte une **Category** et une **Activity**, l'**Initiated by** qui identifie l'acteur (un utilisateur, une application, ou le service lui-même), la cible, et le résultat. Une question du type « qui a supprimé ce secret » se résout par le champ Initiated by.

C'est le journal qui répond à « qui a changé quoi », y compris quand ce changement est la cause d'un incident d'authentification observé ailleurs.

## Provisioning logs

Les Provisioning logs tracent ce que le moteur de provisioning tente vers une application cible, généralement en SCIM.

Deux colonnes se lisent ensemble, et n'en lire qu'une est une source d'erreur.

**Action** décrit l'opération : `Create`, `Update`, `Delete`, `Disable`, `StagedDelete`, `Other`.

**Status** décrit l'issue : `Success`, `Failure`, `Skipped`, `Warning`.

`Skipped` est le statut le plus fréquemment mal interprété. Il ne signale ni erreur ni panne : le moteur a évalué l'utilisateur et décidé de ne rien faire, parce qu'il ne correspond pas au scope filter, parce qu'il n'est pas affecté à l'application, ou parce que rien n'a changé depuis le dernier cycle. Le message d'étape précise la raison. Un scénario du type « cet utilisateur n'apparaît pas dans l'application SaaS alors que le provisioning est en marche et sans erreur » se résout presque toujours par un `Skipped` dû au scope.

Chaque entrée expose quatre étapes : import depuis la source, détermination de la correspondance dans la cible, évaluation du scope, et exécution de l'action. Localiser l'étape en échec oriente directement vers la cause, mapping d'attributs, attribut obligatoire absent, ou correspondance ambiguë.

## Conserver et analyser au-delà du portail

La rétention native est limitée : 7 jours en édition Free, 30 jours en P1 et P2 pour les sign-in et audit logs. Toute exigence de conservation supérieure impose un export.

Les **diagnostic settings**, configurés dans Microsoft Entra ID puis Monitoring puis Diagnostic settings, sélectionnent les catégories de logs à exporter et leur destination. Trois destinations sont à connaître nommément :

| Destination | Usage |
|---|---|
| Log Analytics workspace | Requêtage en KQL, classeurs, alertes, base de Microsoft Sentinel |
| Storage account | Archivage long terme à faible coût, conformité |
| Azure Event Hubs | Diffusion vers un SIEM tiers, Splunk ou QRadar par exemple |

L'export exige au minimum Microsoft Entra ID P1 pour les catégories de logs de connexion.

Une fois les logs dans Log Analytics, KQL permet de les interroger. Les tables utiles sont `SigninLogs`, `AADNonInteractiveUserSignInLogs`, `AADServicePrincipalSignInLogs`, `AADManagedIdentitySignInLogs`, `AuditLogs` et `AADProvisioningLogs`.

```kusto
SigninLogs
| where TimeGenerated > ago(7d)
| where ResultType != 0
| summarize Echecs = count() by UserPrincipalName, ResultType, ResultDescription
| order by Echecs desc
```

## Rapports et classeurs

Le portail expose des analyses prêtes à l'emploi qu'il ne faut pas ignorer, puisque la compétence mesurée les nomme explicitement.

**Usage and insights** donne l'activité par application, les échecs de connexion, et l'usage des méthodes d'authentification.

Les **workbooks** sont des classeurs interactifs adossés à Log Analytics. Les plus utilisés sont Sign-ins, Conditional Access Insights and Reporting, qui sert à mesurer l'effet d'une policy en report-only avant de l'activer, Sensitive Operations Report, et Provisioning Analysis.

**Identity Secure Score** note la posture d'identité du tenant sur un ensemble de contrôles recommandés, avec un pourcentage et des actions d'amélioration. C'est un indicateur de posture et un outil de priorisation, pas un mécanisme de contrôle d'accès : il ne bloque rien.

## Outils de diagnostic intégrés

Deux outils sont à citer, et leur absence dans un raisonnement de dépannage est remarquée.

Le **sign-in diagnostic** s'ouvre depuis un log de connexion, onglet Basic info puis Troubleshoot Event, ou depuis Diagnose and solve problems. Il analyse l'événement et rend une explication en langage clair : quelle policy Conditional Access a bloqué, pourquoi la MFA a été exigée, pourquoi le compte est verrouillé. C'est le premier réflexe sur un échec de connexion, avant toute lecture manuelle.

**Microsoft Entra Connect Health** surveille l'infrastructure hybride : agents de synchronisation, serveurs AD FS, contrôleurs de domaine. Il remonte les erreurs de synchronisation et les alertes d'infrastructure, et exige P1.

## Méthode de dépannage

L'ordre suivant évite de partir dans la mauvaise direction :

1. Ouvrir le sign-in diagnostic sur l'événement en échec, et lire ce qu'il dit avant de raisonner.
2. Identifier l'identité concernée : utilisateur, invité, service principal, managed identity. Cela détermine l'onglet de log.
3. Identifier l'application et la ressource visée, qui peuvent différer.
4. Lire le code d'erreur et sa description. Les codes fréquents valent d'être connus : `50126` mot de passe incorrect, `50053` compte verrouillé ou attaque par pulvérisation, `53003` bloqué par Conditional Access, `50076` MFA requise, `50105` utilisateur non affecté à l'application, `700016` application introuvable dans le tenant, `7000215` secret client invalide.
5. Lire Authentication Details pour comprendre ce qui a été exigé et satisfait.
6. Lire l'onglet Conditional Access pour identifier la policy en `Failure`.
7. Vérifier l'appareil et l'emplacement si la policy en dépend.
8. Si l'incident coïncide avec un changement, corréler avec les Audit logs. Si l'incident concerne un compte absent d'une application cible, aller dans les Provisioning logs.

## Ce qui se joue sur des détails

Il y a exactement quatre catégories de sign-in logs, et les managed identities ont la leur, distincte de celle des service principals.

`Success` dans l'onglet Conditional Access signifie « policy évaluée avec succès », pas « accès accordé par cette policy ».

`Skipped` dans les Provisioning logs n'est pas une erreur : c'est le plus souvent un filtre de scope ou une absence d'affectation.

« MFA requirement satisfied by claim in the token » signifie qu'aucune nouvelle MFA n'a été demandée.

Request ID identifie une requête, Correlation ID regroupe les requêtes d'une même opération.

La rétention native est de 7 jours en Free et 30 jours en P1 et P2. Au-delà, il faut un diagnostic setting vers Log Analytics, un storage account ou un Event Hub.

Identity Secure Score mesure une posture et ne bloque aucun accès.
