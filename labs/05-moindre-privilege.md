# Lab 5. Retrait d'accès et moindre privilège

## Ce que le lab établit

Que retirer un accès demande de distinguer l'identité, l'appartenance et le consentement, et que révoquer un consentement ne coupe pas automatiquement un access token déjà émis.

## Situation de départ

À l'issue des labs précédents, le tenant présente quatre situations à traiter, chacune correspondant à un mécanisme de retrait différent.

Un utilisateur interne détient Reader sur un resource group via son appartenance à un groupe de sécurité. Cet accès est légitime et doit être préservé.

Un invité détient le même accès Azure par le même groupe. Cet accès n'est plus justifié.

Un propriétaire de groupe, qui n'en est pas membre, doit continuer à gérer le groupe sans recevoir ce que le groupe distribue.

Une application détient la permission applicative `User.Read.All` sur Microsoft Graph, consentie au niveau du tenant. Cette permission est trop large pour son besoin réel.

## Prérequis

| Élément | Valeur |
|---|---|
| Rôle pour modifier l'appartenance du groupe | Propriétaire du groupe, ou Groups Administrator |
| Rôle pour révoquer un consentement Graph applicatif | Privileged Role Administrator ou Global Administrator |
| Licence | Aucune |

## Actions et observations

### Retirer l'invité du groupe

Le retrait est immédiat côté annuaire. Côté Azure, la vérification par Check access ne reflète pas forcément le changement instantanément : les caches et la propagation peuvent retarder la prise en compte. Un test effectué trop tôt peut donc donner une lecture trompeuse.

Après propagation, l'invité n'a plus aucun rôle Azure effectif. Son compte, lui, existe toujours dans l'annuaire : l'identité survit à l'entitlement.

Dans un scénario gouverné par Entitlement Management, l'expiration ou le retrait d'une assignment d'access package retire les ressources fournies par cette assignment. Selon la configuration de cycle de vie externe et les autres assignments restantes, le compte invité peut ensuite être nettoyé.

### Vérifier le membre interne

L'utilisateur interne conserve Reader. Le retrait a bien porté sur une appartenance individuelle et non sur l'affectation de rôle, qui vise toujours le groupe.

### Vérifier le propriétaire

Le propriétaire n'a reçu aucun accès Azure à aucun moment du lab. Il continue de gérer la composition du groupe.

C'est ici que le raisonnement de moindre privilège doit aller plus loin que le constat. Le propriétaire peut avoir un chemin d'escalade en s'ajoutant comme membre et en héritant ensuite de ce que le groupe distribue. La propriété d'un groupe est donc un privilège à gouverner, pas un simple rôle décoratif.

### Révoquer le consentement administrateur

La révocation s'effectue depuis Enterprise applications, sur le service principal, onglet Permissions. Elle supprime le grant / app role assignment correspondant.

Un test immédiat montre que l'access token obtenu avant la révocation **continue de fonctionner** contre Microsoft Graph. C'est l'observation la plus utile du lab : l'access token a été émis avec ses autorisations et la suppression ultérieure du consentement ne réécrit pas ce JWT.

Le bon modèle mental est donc :

```text
Révoquer le consentement
→ empêche l'obtention de nouveaux tokens avec cette permission
→ ne détruit pas automatiquement le token déjà émis
```

Même prudence avec la désactivation de l'application ou la suppression du client secret : ces actions empêchent de nouvelles authentifications / émissions de token selon le mécanisme concerné, mais un access token déjà remis peut rester accepté jusqu'à son expiration si la ressource ne dispose pas d'un mécanisme de révocation anticipée applicable à ce scénario.

La **Continuous Access Evaluation** permet à certaines ressources et certains clients compatibles de réagir à des événements critiques dans des scénarios pris en charge, surtout autour des sessions utilisateur. Elle ne doit pas être présentée comme une gomme universelle de tous les tokens app-only.

### Supprimer la permission configurée

La suppression de `User.Read.All` de l'écran API permissions est une opération distincte de la révocation du consentement. Après elle, l'application ne demande plus cette permission dans sa configuration.

![API autorisées après révocation : User.Read reste « Accordé », User.Read.All repasse à « Pas accordé », avec la notification « Autorisation supprimée de SC300-Lab-App »](../screenshots/lab5-consentement-revoque.png)

Les deux permissions se lisent côte à côte : la déléguée conserve son consentement, l'applicative l'a perdu.

L'ordre importe :

- révoquer sans supprimer laisse la permission configurée et donc reconsentable ;
- supprimer la permission configurée sans traiter le grant existant ne doit pas être supposé comme équivalent à une révocation complète de l'autorisation déjà accordée.

### Vérifier les traces

Les Audit logs portent les opérations de configuration, avec l'acteur dans Initiated by : retrait du membre du groupe, retrait du grant / app role assignment et modification des permissions de l'application.

## Ce qu'il faut en retenir

Retirer un accès demande d'identifier **par quel mécanisme il a été accordé**.

- appartenance de groupe → retirer l'appartenance ;
- role assignment Azure → modifier l'affectation RBAC ;
- consentement d'application → révoquer le grant ;
- permission configurée → modifier l'App Registration ;
- identité → désactiver ou supprimer si c'est réellement le besoin.

Révoquer un consentement retire le droit d'obtenir de nouveaux tokens avec ce grant, mais ne garantit pas la disparition immédiate de tous les access tokens déjà émis.

Le moindre privilège consiste à retirer ce qui n'est plus nécessaire, pas tout ce qui est retirable. Le membre interne a conservé son accès parce qu'il était justifié ; l'exercice aurait été raté si le groupe ou l'affectation de rôle avaient été supprimés sans raison.

Ce lab montre en creux la valeur de la gouvernance : ces décisions ont été prises manuellement. Access reviews, access packages et PIM servent précisément à rendre ce cycle plus explicite, révisable et temporaire lorsque les licences et le scénario le permettent.

## Lien avec l'examen

Domaines « Plan and implement workload identities » et « Plan and automate identity governance ». Voir les fiches [03](../fiches/03-identites-workload.md) et [04](../fiches/04-gouvernance-identites.md).
