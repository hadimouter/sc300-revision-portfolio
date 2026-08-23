# Gouvernance des identités

Domaine « Plan and automate identity governance », 20 à 25 % de l'examen.

L'authentification répond à « qui es-tu », l'autorisation à « que peux-tu faire ». La gouvernance répond aux questions que les deux premières laissent ouvertes : pourquoi cet accès existe-t-il, qui l'a validé, pour combien de temps, et qui vérifie qu'il est encore justifié.

## Licences, à traiter en premier

C'est le domaine où les raccourcis de licence vieillissent le plus vite. Il faut distinguer les capacités historiques de Microsoft Entra ID P2 des capacités avancées ajoutées par le produit Microsoft Entra ID Governance.

| Fonctionnalité | Licence à retenir |
|---|---|
| Privileged Identity Management, y compris les capacités PIM historiques | Microsoft Entra ID P2 |
| Access reviews historiquement disponibles en P2 | Microsoft Entra ID P2 |
| Entitlement Management / access packages historiquement disponibles en P2 | Microsoft Entra ID P2 |
| Capacités avancées d'Entitlement Management et d'Access Reviews | Microsoft Entra ID Governance |
| Lifecycle Workflows | Microsoft Entra ID Governance |

Microsoft Entra ID Governance est un produit avancé disponible au-dessus de P1 ou P2. Il contient les capacités historiques de gouvernance de P2 et des capacités supplémentaires. Il est donc **faux** de mémoriser « P2 n'a pas Entitlement Management » : Microsoft indique explicitement que les capacités Entitlement Management et Access Reviews précédemment GA dans P2 restent disponibles avec P2.

Référence : [Microsoft Entra ID Governance licensing fundamentals](https://learn.microsoft.com/en-us/entra/id-governance/licensing-fundamentals).

## Entitlement Management

Entitlement Management industrialise la demande d'accès : au lieu d'ouvrir un ticket pour chaque groupe, l'utilisateur demande un ensemble cohérent, une approbation se déclenche, et l'accès peut expirer automatiquement.

### Les quatre objets

Un **catalog** est un conteneur de ressources. Il n'accorde rien : il délimite ce qui est disponible pour construire des access packages, et permet de déléguer à des propriétaires métier la création de packages sur leur propre périmètre.

Un **access package** regroupe des ressources à obtenir ensemble. Les types de ressources les plus classiques sont des groupes de sécurité ou Microsoft 365, des applications avec service principal, et des sites SharePoint Online.

Pour chaque ressource ajoutée, on choisit un **resource role**. Ce n'est pas un quatrième type de ressource mais le rôle **dans** la ressource : Member ou Owner pour un groupe, l'un des app roles publiés pour une application, l'un des niveaux d'accès SharePoint pour un site.

Une **policy** définit le workflow. Elle répond notamment à quatre questions : qui peut demander, qui approuve, quelles informations sont exigées à la demande, et combien de temps dure l'accès avant expiration ou demande de prolongation. Un package peut porter plusieurs policies, chacune ciblant une population différente.

Une **connected organization** représente un partenaire externe et permet de cibler ses utilisateurs dans une policy. Elle sert à gouverner l'accès d'organisations partenaires sans devoir traiter chaque invité comme une exception isolée.

La chaîne complète est donc : un catalog contient des ressources, un access package en sélectionne certaines avec un rôle, une policy dit qui peut le demander et comment, l'utilisateur demande, l'approbation se joue, une assignment est créée, et l'expiration ou le retrait de cette assignment retire les accès qu'elle fournissait.

### Ce que fait My Access

My Access, à l'adresse `myaccess.microsoft.com`, est le portail utilisateur. On y demande des access packages, on suit ses demandes, on prolonge ses accès, et on répond aux access reviews dont on est relecteur.

Il ne faut pas le confondre avec les portails voisins : `myaccount.microsoft.com` gère le profil et les méthodes d'authentification, `myapps.microsoft.com` liste les applications accessibles, et le centre d'administration Entra sert aux administrateurs.

### Le cas des externes

Un utilisateur externe peut être invité dans le cadre d'un processus d'Entitlement Management et recevoir les ressources du package après approbation. À l'expiration ou au retrait de la dernière assignment, la configuration de gouvernance peut également participer au nettoyage du compte externe.

Le point important est donc :

> Un access package **attribue** des accès via une assignment, et la fin de cette assignment peut aussi **retirer** les ressources accordées.

Il ne faut pas le résumer par « un access package ne retire jamais d'accès ».

## Terms of Use

Le study guide SC-300 actuel nomme explicitement les **Terms of Use (ToU)**.

Les ToU permettent d'exiger qu'un utilisateur accepte des conditions, souvent présentées sous forme de document PDF, avant de poursuivre l'accès. Ils s'intègrent avec **Conditional Access**.

Scénario classique :

> L'entreprise exige que les utilisateurs acceptent un document juridique avant d'accéder à une application.

Réponse : **Terms of Use + policy Conditional Access**.

Ce mécanisme n'attribue pas une ressource comme un access package ; il impose une acceptation avant l'accès.

## Access reviews

Une access review est une recertification : elle demande à quelqu'un de confirmer qu'un accès existant reste justifié. Ce n'est pas un mécanisme initial d'attribution.

### Ce qui peut être revu

Les cibles importantes pour l'examen comprennent notamment :

- l'appartenance à des groupes ;
- l'affectation des utilisateurs à des enterprise applications ;
- les rôles Microsoft Entra via PIM ;
- les rôles Azure resource via PIM ;
- les assignments d'access packages ;
- les utilisateurs invités selon le scénario.

### Les réglages qui font les questions

**Reviewers** peut être les propriétaires du groupe, des utilisateurs désignés, les utilisateurs eux-mêmes en auto-revue, ou le **manager** de chaque personne revue. Un reviewer de secours est utile lorsque certains utilisateurs n'ont pas de manager exploitable.

**Auto apply results to resource** détermine si les décisions sont appliquées automatiquement à la fin de la revue. Désactivé, un administrateur doit appliquer les résultats manuellement.

**If reviewers don't respond** décide du sort des accès non revus, avec des choix tels que No change, Remove access, Approve access ou Take recommendations selon le scénario.

Une revue peut être ponctuelle ou récurrente, et la justification peut être rendue obligatoire.

### Suivi et réponse manuelle

Le study guide demande également de savoir **monitorer** une access review et **répondre manuellement** à une activité de review.

Il faut donc savoir distinguer :

- créer/configurer la review ;
- répondre en tant que reviewer ;
- surveiller son avancement ;
- appliquer les décisions manuellement si l'auto-apply est désactivé.

## Privileged Identity Management

PIM supprime les privilèges permanents en les remplaçant par une éligibilité qu'il faut activer, pour une durée limitée et sous conditions.

### Trois périmètres, pas un

PIM gouverne :

**Les rôles Microsoft Entra**, c'est-à-dire l'administration de l'annuaire.

**Les rôles Azure resource**, c'est-à-dire Azure RBAC sur des management groups, abonnements, resource groups ou ressources.

**Les groupes**, via *PIM for Groups*. On rend un utilisateur éligible à devenir **membre** ou **propriétaire** d'un groupe, puis il active cette appartenance à la demande.

PIM for Groups est particulièrement utile lorsqu'un accès applicatif ou une autre autorisation est distribué par un groupe plutôt que par un rôle Entra/Azure directement.

### Éligible et actif

Une affectation **active** confère les privilèges immédiatement. Une affectation **éligible** ne confère rien tant que l'utilisateur ne l'a pas activée.

Les deux peuvent être permanentes ou limitées dans le temps. L'examen exploite surtout la différence entre privilège permanent actif et éligibilité à activer à la demande.

### Réglages d'activation

Les role settings peuvent imposer notamment :

- durée maximale d'activation ;
- MFA ;
- justification ;
- numéro de ticket ;
- approbation ;
- authentication context Conditional Access à l'activation ;
- notifications.

L'authentication context est la réponse attendue lorsqu'un scénario veut appliquer une exigence Conditional Access renforcée **au moment précis de l'activation PIM**.

### Azure resources, historique et audit

PIM ne concerne pas seulement Entra. Il sait gouverner les rôles Azure RBAC à différents scopes.

L'historique et les rapports PIM servent à analyser les affectations, activations et décisions. Une question du type « qui a activé un rôle privilégié sur les 20 derniers jours » oriente vers l'audit / history PIM, pas vers les sign-in logs.

## Lifecycle Workflows

Lifecycle Workflows automatise les tâches Joiner, Mover, Leaver à partir d'événements et d'attributs du cycle de vie, par exemple une date d'arrivée ou de départ.

C'est une fonction importante de Microsoft Entra ID Governance, mais **elle n'est pas listée comme objectif explicite dans le study guide SC-300 en vigueur depuis le 27 avril 2026**. Elle reste utile pour comprendre l'écosystème IAM et peut apparaître comme sujet connexe, Microsoft précisant que les bullets du study guide ne sont pas exhaustifs.

La confusion à éviter est avec les access reviews : une review demande à un humain si un accès doit rester ; un lifecycle workflow exécute automatiquement des tâches à une étape du cycle de vie.

## Comptes d'urgence

Les comptes break-glass servent à reprendre la main quand tout le reste échoue : panne de fédération, policy Conditional Access mal configurée, problème sur les méthodes d'authentification habituelles.

Points importants :

- disposer d'au moins deux comptes d'urgence ;
- comptes cloud-only, non dépendants d'AD DS/fédération ;
- Global Administrator disponible sans activation PIM ;
- méthodes d'authentification résilientes et distinctes des dépendances ordinaires ;
- exclusions maîtrisées des policies susceptibles de verrouiller le tenant ;
- alerte immédiate sur toute utilisation ;
- validation périodique.

Le principe est de supprimer les dépendances qui pourraient précisément être en panne au moment où le compte d'urgence est nécessaire.

## Ce qui se joue sur des détails

- **P2 inclut encore les capacités historiques d'Entitlement Management et d'Access Reviews** ; Governance ajoute des capacités avancées.
- Lifecycle Workflows exige Microsoft Entra ID Governance.
- Un catalog n'accorde rien par lui-même.
- Un access package accorde des ressources via une assignment et les retire lorsque cette assignment expire ou est retirée.
- Terms of Use s'intègre avec Conditional Access pour exiger une acceptation avant l'accès.
- PIM couvre les rôles Entra, les rôles Azure et les groupes.
- Une access review avec `Auto apply results` désactivé ne modifie pas la ressource tant qu'un administrateur n'applique pas les décisions.
- `If reviewers don't respond` décide du sort des accès non revus.
- Un compte break-glass doit rester utilisable même quand PIM ou une policy ordinaire ne l'est plus.
- Une access review recertifie un accès existant ; elle ne remplace ni Entitlement Management ni un workflow de cycle de vie.
