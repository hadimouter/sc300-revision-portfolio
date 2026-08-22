# Pièges d'examen

Ces confusions ne sont pas des subtilités de vocabulaire : ce sont les endroits précis où une réponse plausible est fausse. Chacune est formulée comme une affirmation vraie, suivie de ce qui la rend piégeuse.

## Applications et permissions

**Configurer une permission ne l'accorde pas.** Ajouter `User.Read.All` dans API permissions laisse un avertissement dans la colonne Status et l'appel échoue. Tant que le consentement n'est pas accordé, rien ne fonctionne. Ce sont deux opérations distinctes, et les retirer sont également deux opérations distinctes.

**Une permission applicative exige toujours un consentement administrateur.** Il n'y a aucune exception, quels que soient les réglages de consentement du tenant. Le mot « généralement » est un distracteur.

**Application Administrator ne peut pas consentir aux app roles Microsoft Graph.** C'est la seule famille de permissions que ce rôle, et Cloud Application Administrator, ne peuvent pas accorder. Il faut Privileged Role Administrator ou Global Administrator. Une question de moindre privilège sur un consentement Graph applicatif attend cette réponse.

**Le nom d'une permission ne suffit jamais.** `User.Read.All` existe en délégué et en applicatif. En délégué, les droits effectifs sont l'intersection avec ceux de l'utilisateur ; en applicatif, la permission s'applique à tout le tenant.

**`roles` n'est pas réservé aux tokens app-only.** Dans un token utilisateur, il porte les app roles métier assignés à cet utilisateur ou à ses groupes. Seul `idtyp` valant `app` identifie de façon fiable un token app-only. `scp`, lui, n'apparaît bien que dans les tokens délégués.

**`.default` est obligatoire dans le client credentials flow.** Ce n'est pas une commodité : demander une permission individuelle provoque une erreur. Et le contenu du token dépend de ce qui a été consenti, pas de ce que le code demande.

**Révoquer un consentement ne coupe pas les tokens déjà émis.** Ils restent valides jusqu'à expiration, environ une heure. Pour couper immédiatement, il faut désactiver le service principal ou révoquer les sessions.

**Assignment required n'accorde aucun consentement.** Il restreint qui peut obtenir un token. En revanche, dans l'autre sens, activer l'affectation rend le consentement utilisateur en libre-service insuffisant, puisque l'absence d'affectation bloque en amont.

**L'affectation par groupe à une enterprise application exige P1.** En édition Free, seules les affectations individuelles fonctionnent.

**Un application object et un service principal ont le même Client ID et des Object ID différents.** C'est le test qui prouve qu'il s'agit de deux objets.

**Une managed identity ne dispense pas d'autorisation.** Elle supprime le credential, pas le besoin d'app roles Graph consentis sur son service principal.

**Une managed identity ne fonctionne que sur Azure.** Pour un workload hors Azure, la réponse est la workload identity federation.

**Le proxy d'application exige P1**, et seule la préauthentification Microsoft Entra ID permet d'y appliquer le Conditional Access. En mode Passthrough, aucune policy CA ne s'applique.

## Authentification et Conditional Access

**La MFA est un grant control, jamais une condition.** Une condition décrit le contexte, un grant control décrit l'exigence.

**L'emplacement réseau n'est plus une condition.** Dans le portail actuel, Network est un bloc distinct au sein d'Assignments, aux côtés de Users, Target resources et Conditions.

**« Require authentication strength » et « Require multifactor authentication » sont mutuellement exclusifs** dans une même policy Conditional Access.

**Security Defaults et Conditional Access s'excluent techniquement.** Ce n'est pas une recommandation : tant que les Security Defaults sont actifs, la création d'une policy CA est refusée.

**Une méthode activée n'est pas une méthode enregistrée.** Le périmètre autorisé par la policy et le périmètre réellement enregistré par les utilisateurs sont deux choses différentes, et l'écart se mesure dans le rapport User registration details.

**Un TAP est créé par un administrateur pour quelqu'un d'autre.** Jamais pour soi-même. Le rôle est Authentication Administrator, ou Privileged Authentication Administrator pour agir sur des comptes administrateurs.

**Le caractère à usage unique d'un TAP se décide à la création de chaque pass.** La policy ne fait que plafonner ce choix. Une policy dont One-time use est à False autorise les deux ; à True, elle impose l'usage unique.

**Un TAP satisfait la MFA mais aucune authentication strength résistante au phishing.**

**« MFA requirement satisfied by claim in the token » signifie qu'aucune nouvelle MFA n'a été demandée.** L'exigence a été satisfaite par un claim antérieur. C'est le fonctionnement normal du SSO, pas une anomalie.

**Une session policy Defender for Cloud Apps ne fonctionne pas seule.** Comme une access policy, elle exige que l'application soit routée par une policy Conditional Access dont le session control est Conditional Access App Control.

**Les policies de risque exigent P2.** Un tenant en P1 ne peut pas exploiter le risque utilisateur ni le risque de connexion, quelle que soit la formulation du scénario.

**Le risque utilisateur porte sur le compte, le risque de connexion sur une tentative.** La remédiation diffère : changement de mot de passe sécurisé dans un cas, MFA dans l'autre.

**« Compliant : No » ne signifie ni appareil compromis ni appareil non géré.** Un appareil peut être managed sans être compliant, par exemple faute d'une mise à jour.

**L'emplacement affiché dans un sign-in log n'est pas une named location.** Le premier est une donnée déduite de l'adresse IP, le second est un objet de configuration.

## Identités et annuaire

**Global Administrator n'est pas Owner des abonnements Azure.** Mais il peut basculer « Access management for Azure resources » et recevoir User Access Administrator à la portée racine `/`. Pas Owner, pas automatique, et tracé dans les Audit logs.

**Un tenant n'est pas un abonnement.** Un abonnement fait confiance à un seul tenant, un tenant peut servir plusieurs abonnements ou aucun.

**Un rôle Entra n'est pas un rôle Azure RBAC.** Deux systèmes séparés, deux portées différentes, aucune propagation de l'un vers l'autre.

**Accepter une invitation ne transforme pas un Guest en Member.** Cela fait passer `externalUserState` à `Accepted`. Le changement de `UserType` est une opération distincte et manuelle.

**`UserType` décrit la relation, pas la provenance.** Il existe des internal guests et des external members ; la cross-tenant synchronization produit par défaut des external members, pas des invités.

**Un Owner de groupe n'est pas Member.** Il ne reçoit ni licence, ni rôle Azure RBAC, ni accès applicatif attribué au groupe. Mais il peut s'auto-ajouter comme membre : c'est un chemin d'escalade à voir dans tout raisonnement de moindre privilège.

**Un groupe role-assignable ne peut pas être dynamique**, et la propriété `isAssignableToRole` se fixe définitivement à la création.

**Une administrative unit n'est pas transitive.** Y placer un groupe donne le droit de gérer ce groupe, pas ses membres.

**Une restricted management AU bloque même les administrateurs à portée tenant**, à l'exception du Global Administrator.

**Une licence sans `usageLocation` échoue**, y compris via un groupe, et l'erreur n'apparaît que dans les propriétés de licence de l'utilisateur.

**Une licence héritée d'un groupe ne se retire pas individuellement.** Il faut sortir l'utilisateur du groupe.

**Cross-tenant access n'est pas cross-tenant synchronization.** Le premier définit une relation de confiance et des règles d'accès, le second provisionne des comptes.

**Le discriminant entre Connect Sync et Cloud Sync n'est pas le nombre de forêts.** Les deux gèrent plusieurs forêts. Cloud Sync est le seul à gérer des forêts déconnectées ; Connect Sync est le seul à synchroniser les appareils, donc à permettre le Hybrid Entra join.

**Seamless SSO exige un appareil joint au domaine Active Directory.** Il ne concerne pas les appareils Entra joined ou hybrid joined, qui obtiennent le SSO par leur Primary Refresh Token.

**PHS n'est pas PTA.** PHS fait authentifier dans le cloud et survit à une coupure du site local ; PTA valide contre l'Active Directory local et dépend de la disponibilité des agents.

**PHS est nécessaire à la détection d'identifiants divulgués** d'ID Protection. Sans elle, cette détection ne peut pas fonctionner.

## Gouvernance

**Entitlement Management et Lifecycle Workflows exigent Microsoft Entra ID Governance**, une référence additionnelle. Un tenant en P2 ne les a pas. C'est le piège de licence le plus rentable du domaine.

**Un access package n'est pas une access review.** L'un fait obtenir, l'autre fait conserver ou retirer. Aucun ne fait le travail de l'autre.

**Une access review n'est pas un lifecycle workflow.** La revue pose une question à un humain, périodiquement ; le workflow exécute des actions automatiquement à une étape du cycle de vie.

**Un catalog n'accorde rien.** Il délimite ce qui est disponible pour construire des packages.

**Un resource role n'est pas un type de ressource.** Les types sont au nombre de trois : groupes, applications, sites SharePoint. Le resource role est le rôle dans la ressource.

**Une access review dont « Auto apply results to resource » est désactivé ne change rien** tant qu'un administrateur n'applique pas les décisions. C'est la cause la plus fréquente de « la revue est finie mais l'accès est toujours là ».

**« If reviewers don't respond » décide du sort des accès non revus**, avec quatre valeurs : No change, Remove access, Approve access, Take recommendations.

**Éligible ne signifie pas actif.** Une affectation éligible ne confère aucun privilège tant qu'elle n'est pas activée.

**PIM ne gouverne pas que les rôles.** Il couvre les rôles Entra, les rôles Azure et les groupes. Un accès applicatif distribué par groupe ne peut être rendu temporaire que par PIM for Groups.

**PIM ne remplace pas le Conditional Access**, et une policy CA ne peut pas cibler directement une activation PIM : il faut passer par un authentication context déclaré dans les role settings.

**Un compte break-glass reçoit son rôle de façon permanente, pas via PIM.** L'activation pourrait être précisément ce qui est cassé. Et il est exclu des policies CA, ce qui rend la surveillance de son usage obligatoire.

## Journaux

**Il y a exactement quatre catégories de sign-in logs.** Les managed identities ont la leur, distincte de celle des service principals. Une question sur une Azure Function ou une VM Azure attend l'onglet Managed identity sign-ins.

**`Success` dans l'onglet Conditional Access signifie « policy évaluée avec succès »**, pas « accès accordé par cette policy ». Une connexion bloquée présente au moins une policy en `Failure`. Une policy en `Not applied` signale un problème de ciblage.

**`Skipped` dans les Provisioning logs n'est pas une erreur.** C'est le plus souvent un filtre de scope ou une absence d'affectation à l'application. C'est la réponse à « le provisioning tourne sans erreur mais l'utilisateur n'apparaît pas ».

**Request ID et Correlation ID ne servent pas à la même chose.** Le premier identifie une requête unique, le second regroupe les requêtes d'une même opération.

**La rétention native est de 7 jours en Free et 30 jours en P1 et P2.** Toute exigence supérieure impose un diagnostic setting vers Log Analytics, un storage account ou un Event Hub.

**Identity Secure Score ne bloque rien.** C'est un indicateur de posture, pas un contrôle d'accès.

**Un code d'erreur se lit avec son contexte.** `50053` peut signaler un compte verrouillé légitimement comme une attaque par pulvérisation de mots de passe ; `50105` signale une absence d'affectation à l'application et non un problème d'authentification.
