# Gouvernance des identités

Domaine « Plan and automate identity governance », 20 à 25 % de l'examen.

L'authentification répond à « qui es-tu », l'autorisation à « que peux-tu faire ». La gouvernance répond aux questions que les deux premières laissent ouvertes : pourquoi cet accès existe-t-il, qui l'a validé, pour combien de temps, et qui vérifie qu'il est encore justifié.

## Licences, à traiter en premier

C'est le domaine où la licence discrimine le plus de réponses, et le seul où P2 ne suffit pas toujours.

| Fonctionnalité | Licence requise |
|---|---|
| Privileged Identity Management, y compris PIM for Groups | Microsoft Entra ID P2 |
| Access reviews | Microsoft Entra ID P2 |
| Entitlement Management, access packages | Microsoft Entra ID Governance |
| Lifecycle Workflows | Microsoft Entra ID Governance |
| Provisioning piloté par les RH, écriture vers AD | Microsoft Entra ID Governance |
| Expiration et revue d'accès des invités | P2 pour les revues, Governance pour les access packages |

Microsoft Entra ID Governance est une référence **additionnelle**, vendue par-dessus P1 ou P2. Un tenant en P2 n'a donc ni Entitlement Management ni Lifecycle Workflows. Un énoncé qui précise l'édition tranche à lui seul plusieurs distracteurs.

## Entitlement Management

Entitlement Management industrialise la demande d'accès : au lieu d'ouvrir un ticket pour chaque groupe, l'utilisateur demande un ensemble cohérent, une approbation se déclenche, et l'accès expire tout seul.

### Les quatre objets

Un **catalog** est un conteneur de ressources. Il n'accorde rien : il délimite ce qui est disponible pour construire des access packages, et permet de déléguer à des propriétaires métier la création de packages sur leur propre périmètre. Un catalog peut être marqué comme activé pour les utilisateurs externes.

Un **access package** regroupe des ressources à obtenir ensemble. Les types de ressources sont au nombre de trois : des groupes de sécurité ou Microsoft 365, des applications avec service principal, et des sites SharePoint Online.

Pour chaque ressource ajoutée, on choisit un **resource role**. Ce n'est pas un quatrième type de ressource mais le rôle **dans** la ressource : Member ou Owner pour un groupe, l'un des app roles publiés pour une application, l'un des niveaux d'accès SharePoint pour un site. Une même ressource peut ainsi apparaître dans deux packages avec des rôles différents.

Une **policy** définit le workflow. Elle répond à quatre questions : qui peut demander (des utilisateurs internes, les membres de connected organizations, tout utilisateur externe, ou personne dans le cas d'une affectation directe par un administrateur), qui approuve (jusqu'à deux étapes, avec approbateurs de secours et délais), quelles informations sont exigées à la demande, et combien de temps dure l'accès avant expiration ou demande de prolongation. Un package peut porter plusieurs policies, chacune ciblant une population différente.

Une **connected organization** représente un partenaire externe et permet de cibler ses utilisateurs dans une policy. Son identité peut provenir de quatre sources : un tenant Microsoft Entra dans n'importe quel cloud Microsoft, un fournisseur d'identité tiers fédéré en SAML ou WS-Fed, un domaine de messagerie pour lequel le passage par one-time passcode est accepté, ou un tenant Azure AD B2C.

La chaîne complète est donc : un catalog contient des ressources, un access package en sélectionne certaines avec un rôle, une policy dit qui peut le demander et comment, l'utilisateur demande, l'approbation se joue, une assignment est créée, et l'expiration la retire.

### Ce que fait My Access

My Access, à l'adresse `myaccess.microsoft.com`, est le portail utilisateur. On y demande des access packages, on suit ses demandes, on prolonge ses accès, et on répond aux access reviews dont on est relecteur.

Il ne faut pas le confondre avec les portails voisins : `myaccount.microsoft.com` gère le profil et les méthodes d'authentification, `myapps.microsoft.com` liste les applications accessibles, et le centre d'administration Entra sert aux administrateurs. Une question de scénario qui demande « où l'utilisateur externe demande-t-il son accès » attend My Access.

### Le cas des externes

C'est le mécanisme le plus élégant du produit et il est souvent testé. Un utilisateur externe qui n'existe pas encore dans l'annuaire peut demander un access package via un lien My Access. S'il est approuvé, Entra **l'invite automatiquement** comme utilisateur B2B et lui accorde l'accès dans la foulée. Quand son assignment expire ou lui est retirée, la policy peut déclencher la **suppression du compte invité** s'il ne détient plus aucune autre assignment.

L'identité et l'accès sont ainsi créés et détruits ensemble, ce qui répond à la question du nettoyage des invités orphelins sans script ni revue manuelle.

## Access reviews

Une access review est une recertification : elle demande à quelqu'un de confirmer qu'un accès existant reste justifié. Ce n'est pas un mécanisme d'attribution.

### Ce qui peut être revu

L'inventaire complet compte plus de cibles que ce qu'on retient spontanément :

- l'appartenance à des groupes de sécurité ou Microsoft 365, y compris les membres éligibles quand le groupe est géré par PIM for Groups
- l'affectation des utilisateurs à des enterprise applications
- les rôles Microsoft Entra, actifs et éligibles, via PIM
- les rôles Azure resource, actifs et éligibles, via PIM
- les assignments d'access packages
- les utilisateurs invités, sur l'ensemble du tenant
- les service principals affectés à des rôles privilégiés, avec Workload ID Premium

### Les réglages qui font les questions

La définition d'une revue est rarement testée. Ses paramètres de fin le sont systématiquement.

**Reviewers** peut être les propriétaires du groupe, des utilisateurs désignés, les utilisateurs eux-mêmes en auto-revue, ou le **manager** de chaque personne revue, auquel cas un relecteur de secours doit être désigné pour les comptes sans manager.

**Auto apply results to resource** détermine si les décisions sont appliquées automatiquement à la fin de la revue. Désactivé, un administrateur doit appliquer manuellement, et rien ne change tant qu'il ne l'a pas fait. C'est la cause la plus fréquente de « la revue est terminée mais l'accès est toujours là ».

**If reviewers don't respond** décide du sort des accès non revus, avec quatre valeurs : No change, Remove access, Approve access, ou Take recommendations. C'est le réglage à citer quand un scénario demande de retirer automatiquement les accès sur lesquels personne ne s'est prononcé.

Les **recommandations** proposent une décision à partir de la dernière connexion, avec un seuil d'inactivité configurable, par défaut 30 jours.

Une revue peut être ponctuelle ou récurrente, et la justification peut être rendue obligatoire.

## Privileged Identity Management

PIM supprime les privilèges permanents en les remplaçant par une éligibilité qu'il faut activer, pour une durée limitée et sous conditions.

### Trois périmètres, pas un

C'est l'omission qui coûte le plus cher sur ce sujet. PIM gouverne :

**Les rôles Microsoft Entra**, c'est-à-dire l'administration de l'annuaire.

**Les rôles Azure resource**, c'est-à-dire Azure RBAC sur des management groups, abonnements, resource groups ou ressources.

**Les groupes**, via *PIM for Groups*. On rend un utilisateur éligible à devenir **membre** ou **propriétaire** d'un groupe, et il active cette appartenance à la demande. C'est la seule réponse possible quand un scénario demande une élévation temporaire vers quelque chose que PIM ne gouverne pas directement : un accès applicatif distribué par groupe, une licence, un rôle dans une application SaaS. Le groupe doit être role-assignable pour porter des rôles Entra.

### Éligible et actif

Une affectation **active** confère les privilèges immédiatement. Une affectation **éligible** ne confère rien tant que l'utilisateur ne l'a pas activée. Les deux peuvent être permanentes ou limitées dans le temps, ce qui donne quatre combinaisons ; l'examen exploite surtout la différence entre « permanent active », le privilège permanent qu'on cherche à éliminer, et « éligible », l'activation à la demande.

L'activation se fait dans PIM, ou depuis le bandeau du centre d'administration, et se journalise dans les Audit logs.

### Réglages d'activation

Les role settings, définis rôle par rôle, exposent davantage de contrôles que la liste habituelle :

- durée maximale d'activation, de 1 à 24 heures
- exigence de MFA à l'activation
- exigence d'une **justification** écrite
- exigence d'un **numéro de ticket** et d'un système de ticket, ce qui permet de tracer vers l'outil ITSM
- exigence d'**approbation**, avec des approbateurs désignés
- exigence d'un **authentication context** Conditional Access à l'activation, ce qui permet d'imposer par exemple une MFA résistante au phishing ou un appareil conforme au moment précis de l'élévation
- notifications aux administrateurs, aux approbateurs et à l'activateur

L'authentication context est la réponse attendue quand un scénario veut appliquer des conditions Conditional Access à l'activation d'un rôle, puisqu'une policy CA classique ne peut pas cibler une activation PIM directement.

### Alertes et revues

PIM produit ses propres alertes : trop d'administrateurs globaux, rôles attribués hors de PIM, comptes n'utilisant jamais leur rôle, activations trop fréquentes. Il porte également ses propres access reviews de rôles, capables de couvrir les affectations actives comme éligibles.

## Lifecycle Workflows

Lifecycle Workflows automatise les tâches liées aux étapes de la vie professionnelle d'un utilisateur. Le vocabulaire officiel est **Joiner, Mover, Leaver**.

Un workflow se compose de deux parties. Les **execution conditions** définissent le périmètre, c'est-à-dire la population concernée par une règle sur les attributs, et le déclencheur, exprimé en jours avant ou après une date. Les **tasks** définissent les actions, choisies dans une bibliothèque d'une trentaine de tâches intégrées : envoyer un courriel de bienvenue au manager, générer un Temporary Access Pass, ajouter à des groupes ou à des équipes, désactiver le compte, retirer de tous les groupes, retirer toutes les licences, supprimer le compte, ou déclencher un flux Logic Apps personnalisé.

Les déclencheurs reposent sur les attributs `employeeHireDate` et `employeeLeaveDateTime`, qui doivent donc être alimentés, généralement par le connecteur RH (Workday, SuccessFactors) ou par la synchronisation depuis l'Active Directory.

La confusion à éviter est avec les access reviews. Une access review demande « cet accès est-il encore justifié », de manière périodique et avec un humain qui décide. Un lifecycle workflow exécute « fais ces actions à cette étape », de manière automatique et sans décision humaine.

## Comptes d'urgence

Les comptes break-glass servent à reprendre la main quand tout le reste échoue : panne de la fédération, policy Conditional Access mal configurée qui verrouille tous les administrateurs, indisponibilité d'un fournisseur de MFA.

La checklist Microsoft est précise, et c'est elle qui est testée plutôt que le principe :

- au moins **deux** comptes, pour couvrir la défaillance de l'un
- **cloud-only**, sur le domaine `*.onmicrosoft.com`, jamais fédérés ni synchronisés depuis un annuaire local
- non associés à une personne physique ni à un téléphone ou un appareil individuel
- rôle **Global Administrator** attribué de façon **permanente**, et non via une éligibilité PIM, puisque l'activation pourrait elle-même être bloquée
- identifiants résistants au phishing, avec au moins un compte exclu de toute policy Conditional Access qui pourrait le bloquer
- identifiants stockés physiquement, séparés, en lieu sûr
- surveillance des connexions avec **alerte immédiate** sur toute utilisation, via Log Analytics ou Sentinel
- validation périodique, au moins tous les 90 jours et après tout changement majeur de configuration

Exclure un compte de toutes les policies CA est un compromis assumé : c'est précisément ce qui le rend utilisable quand une policy verrouille le tenant, et c'est pourquoi la surveillance de son usage est obligatoire.

## Ce qui se joue sur des détails

Entitlement Management et Lifecycle Workflows exigent Microsoft Entra ID Governance, pas seulement P2.

Un access package accepte trois types de ressources : groupes, applications, sites SharePoint. Le resource role est le rôle dans la ressource, pas un type de ressource.

Un catalog n'accorde rien par lui-même.

PIM couvre les rôles Entra, les rôles Azure et les groupes. PIM for Groups est la seule voie pour rendre temporaire un accès distribué par groupe.

Une access review dont « Auto apply results » est désactivé ne change rien tant qu'un administrateur n'applique pas les décisions.

« If reviewers don't respond » est le réglage qui décide du sort des accès non revus.

Un compte break-glass reçoit son rôle de façon permanente et non via PIM, parce que l'activation peut être précisément ce qui est cassé.

Une access review recertifie un accès existant ; un lifecycle workflow exécute des actions à une étape du cycle de vie. Aucun des deux ne remplace l'autre.
