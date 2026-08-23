# Pièges d'examen

Ces confusions ne sont pas des subtilités de vocabulaire : ce sont les endroits précis où une réponse plausible est fausse. Chacune est formulée comme une affirmation vraie, suivie de ce qui la rend piégeuse.

## Applications et permissions

**Configurer une permission ne l'accorde pas.** Ajouter `User.Read.All` dans API permissions ne vaut pas consentement. La configuration et le grant sont deux opérations distinctes.

**Une permission applicative exige un consentement administrateur.** Le mot « généralement » est un mauvais raccourci ici.

**Application Administrator ne peut pas consentir aux app roles Microsoft Graph.** Pour cette famille particulièrement sensible, il faut un rôle plus privilégié comme Privileged Role Administrator ou Global Administrator.

**Le nom d'une permission ne suffit jamais.** `User.Read.All` existe en délégué et en applicatif. Le type détermine le contexte et la portée réelle.

**`roles` n'est pas réservé aux tokens app-only.** Un token utilisateur peut aussi porter des app roles métier dans `roles`. `idtyp=app` est un marqueur fiable d'un token app-only ; `scp` identifie les permissions déléguées.

**`.default` est obligatoire dans le client credentials flow v2.0 pour Microsoft Graph.** Le token reflète les permissions applicatives déjà configurées et consenties pour la ressource.

**Révoquer un consentement ne fait pas disparaître les access tokens déjà émis.** Ils peuvent rester valides jusqu'à expiration ou jusqu'à ce qu'un mécanisme pris en charge par la ressource les rejette. Désactiver une application ou supprimer son credential empêche surtout de nouvelles émissions de token ; cela ne voyage pas dans le temps pour effacer un JWT déjà remis.

**Assignment required n'accorde aucun consentement.** Il contrôle qui peut obtenir un token / accéder à l'application selon l'affectation. Affectation et consentement répondent à deux questions différentes.

**L'affectation par groupe à une enterprise application peut avoir un prérequis de licence supérieur à l'affectation individuelle.** Lire l'édition précisée dans l'énoncé.

**Un application object et un service principal ont le même Client ID et des Object ID différents.** C'est le test le plus simple pour vérifier qu'on regarde deux objets distincts.

**Une managed identity ne dispense pas d'autorisation.** Elle supprime le problème du credential, pas le besoin d'un rôle Azure RBAC ou d'une permission Graph selon la ressource cible.

**Une managed identity concerne les workloads pris en charge par Azure.** Pour un workload externe disposant d'un IdP OIDC, penser workload identity federation.

**Un managed service account n'est pas une managed identity.** MSA/gMSA appartient au monde Windows / AD DS ; managed identity appartient au monde Azure.

## Authentification et Conditional Access

**La MFA est un grant control, pas une condition.** Une condition décrit le contexte ; un grant control décrit ce qui doit être satisfait pour obtenir l'accès.

**Network est un bloc distinct dans le portail Conditional Access actuel.** Ne pas raisonner uniquement avec d'anciennes captures où l'emplacement était rangé sous Conditions.

**« Require authentication strength » et « Require multifactor authentication » ne sont pas deux cases à empiler sans réfléchir.** Une authentication strength exprime déjà les combinaisons de méthodes acceptables.

**Security Defaults et Conditional Access personnalisé ne s'utilisent pas comme deux couches indépendantes.** Un tenant qui bascule vers des policies CA personnalisées doit traiter explicitement les Security Defaults.

**Une méthode activée n'est pas une méthode enregistrée.** La policy autorise ; l'utilisateur doit encore enregistrer réellement sa méthode.

**Un TAP est créé par un administrateur pour un autre utilisateur.** Il sert à l'onboarding / recovery, pas à se fabriquer soi-même une porte de secours.

**Le caractère one-time d'un TAP dépend de sa configuration de policy et de la création du pass.** Lire les deux niveaux de configuration au lieu de supposer qu'un TAP est toujours mono-usage.

**Un TAP peut satisfaire une exigence MFA sans être une méthode phishing-resistant.** Ne pas confondre « MFA » et « phishing-resistant MFA ».

**« MFA requirement satisfied by claim in the token » signifie qu'aucune nouvelle MFA n'a été demandée pour cet événement.** L'exigence a été satisfaite à partir d'une preuve déjà présente dans le contexte de session/token.

**CAE ne signifie pas “tous les tokens sont supprimés instantanément”.** Sur des clients et ressources compatibles, certains événements critiques comme la désactivation d'un compte peuvent entraîner le rejet d'un token avant son expiration normale. Les capacités CAE liées aux critical events ne doivent pas être réduites au raccourci « P1 obligatoire ».

**Une session policy Defender for Cloud Apps ne fonctionne pas seule.** Le trafic doit être routé vers Conditional Access App Control pour le contrôle en temps réel par reverse proxy.

**Les policies de risque utilisateur / sign-in risk avancées exigent P2.** Ne pas confondre visibilité de base et remédiation/policy basée sur le risque.

**Le risque utilisateur porte sur le compte, le risque de connexion sur une tentative.** C'est la distinction qui tranche la majorité des scénarios.

**`Compliant: No` ne signifie ni compromis ni non géré.** Managed et compliant sont deux états différents.

**L'emplacement affiché dans un sign-in log n'est pas une Named Location.** Le premier est une observation ; la seconde est un objet de configuration.

**CBA high-affinity : penser SKI/X509SKI.** Une question de binding fort ne demande pas simplement « UPN » par réflexe.

**Windows Hello for Business en hybride : reconnaître Cloud Kerberos Trust.** Le PIN n'est pas le secret envoyé au serveur ; il déverrouille une clé protégée sur l'appareil.

## Identités et annuaire

**Global Administrator n'est pas Owner des abonnements Azure.** Le commutateur `Access management for Azure resources` lui permet d'obtenir User Access Administrator à la portée racine, ce qui est autre chose qu'Owner.

**Un tenant n'est pas un abonnement.** Un tenant peut exister sans abonnement et servir plusieurs abonnements.

**Un rôle Entra n'est pas un rôle Azure RBAC.** Deux systèmes séparés, deux portées différentes.

**Accepter une invitation ne transforme pas un Guest en Member.** Cela change l'état d'acceptation, pas automatiquement `UserType`.

**`UserType` décrit la relation à l'organisation, pas l'endroit où l'identité s'authentifie.**

**Un Owner de groupe n'est pas Member.** Il ne reçoit pas automatiquement les accès distribués au groupe, même s'il peut avoir la capacité de modifier sa composition.

**Un groupe role-assignable ne peut pas être dynamique.** Sa propriété de rôle doit être pensée dès sa création.

**Une administrative unit n'est pas transitive.** Placer un groupe dans une AU ne place pas automatiquement les membres de ce groupe dans le scope utilisateur de l'AU.

**Une licence sans `usageLocation` peut échouer.** Le contexte de licence et les propriétés de l'utilisateur comptent.

**Une licence héritée d'un groupe se gère au niveau du mécanisme qui l'a attribuée.** Ne pas tenter de retirer individuellement un héritage de groupe comme s'il s'agissait d'une affectation directe.

**Cross-tenant access n'est pas cross-tenant synchronization.** Le premier définit règles/trust ; le second provisionne des comptes.

**Le discriminant entre Connect Sync et Cloud Sync n'est pas “multi-forêts”.** Regarder plutôt les besoins comme forêts déconnectées, appareils / hybrid join, scénarios Exchange hybrides et possibilités de synchronisation.

**PHS n'est pas PTA.** PHS authentifie côté cloud avec un dérivé synchronisé ; PTA valide le mot de passe contre AD DS via les agents.

**Custom domain : ajouter n'est pas vérifier.** La preuve de propriété DNS attend typiquement **TXT ou MX**.

**Bulk B2B : connaître les colonnes exactes.** `inviteeEmail` et `inviteRedirectUrl` sont des détails de fichier CSV faciles à rater.

## Gouvernance

**P2 n'est pas dépourvu d'Entitlement Management.** Microsoft indique que les capacités Entitlement Management et Access Reviews historiquement GA dans P2 restent incluses en P2. Entra ID Governance ajoute des capacités avancées. Le vieux raccourci « access package = Governance obligatoire dans tous les cas » n'est plus fiable.

**Lifecycle Workflows exige Microsoft Entra ID Governance.** C'est une capacité avancée et, dans le study guide SC-300 du 27 avril 2026, ce n'est pas un bullet explicite même si le sujet reste connexe à l'IAM.

**Un access package n'est pas une access review.** Le package gère une assignment de ressources avec demande/approbation/durée ; la review recertifie un accès existant.

**Un access package peut retirer les accès de son assignment à l'expiration ou au retrait.** Donc « un access package ne retire jamais » est faux.

**Un catalog n'accorde rien.** Il contient ce qui peut être utilisé pour construire les packages.

**Un resource role n'est pas un type de ressource.** C'est le rôle choisi dans une ressource incluse au package.

**Terms of Use n'est pas Entitlement Management.** Pour imposer l'acceptation d'un PDF / de conditions avant l'accès, penser Terms of Use + Conditional Access.

**Une access review avec Auto apply désactivé ne modifie pas automatiquement la ressource.** Il faut appliquer les résultats manuellement.

**`If reviewers don't respond` décide du sort des accès non revus.** C'est différent de la décision explicite d'un reviewer.

**Éligible ne signifie pas actif dans PIM.** Une éligibilité ne donne pas le privilège tant qu'elle n'est pas activée.

**PIM couvre les rôles Entra, les rôles Azure et les groupes.** Ne pas limiter PIM au portail Entra roles.

**L'historique PIM est le bon endroit pour les activations/assignments privilégiés.** Ne pas tout envoyer aux sign-in logs.

**Un compte break-glass ne doit pas dépendre du mécanisme qui pourrait être en panne.** C'est le principe qui guide le choix de ses méthodes d'authentification, de son rôle et de ses exclusions.

## Enterprise applications et Defender for Cloud Apps

**SAML claims se configurent dans le SSO de l'Enterprise Application, pas dans API permissions.** Les permissions OAuth/Graph et les assertions SAML sont deux mondes différents.

**Changer un mapping SCIM = Audit logs ; échec d'exécution SCIM = Provisioning logs.** Le premier répond à « qui a changé », le second à « qu'a fait le moteur ».

**Cloud Discovery n'est pas OAuth app governance.** Cloud Discovery vise le Shadow IT / l'usage des applications ; les OAuth app policies gouvernent les applications OAuth connectées.

**Cloud App Catalog n'est pas My Apps.** Le premier contient les informations et scores de risque des apps cloud ; My Apps est une expérience utilisateur d'accès aux applications.

**Access policy et session policy ne répondent pas à la même question.** Access décide d'entrer ou non ; session contrôle ce qui se passe pendant la session.

## Journaux

**Il y a quatre catégories importantes de sign-in logs.** Utilisateurs interactive, utilisateurs non-interactive, service principals, managed identities.

**`Success` dans l'onglet Conditional Access signifie que la policy applicable a été satisfaite, pas qu'elle est l'unique raison de l'accès.**

**`Skipped` dans Provisioning logs n'est pas nécessairement une erreur.** Lire le scope et le détail de l'étape.

**Request ID et Correlation ID n'ont pas le même rôle.** Le premier identifie une requête ; le second aide à regrouper une opération.

**Au-delà de la rétention native, il faut exporter.** Diagnostic settings vers Log Analytics, Storage ou Event Hub selon l'objectif.

**Identity Secure Score ne bloque rien.** C'est un indicateur de posture.

**Un code d'erreur se lit avec son contexte.** Le numéro seul ne remplace pas Authentication Details, Conditional Access et les informations de l'événement.
