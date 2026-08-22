# Authentification et gestion des accès

Domaine « Implement authentication and access management », 25 à 30 % de l'examen. C'est le domaine le plus lourd et celui qui concentre le plus de questions de scénario.

La logique d'ensemble tient en trois couches. La **policy de méthodes d'authentification** décide de ce qu'un utilisateur a le droit d'utiliser pour prouver son identité. L'**authentication strength** décide de ce qui est assez fort dans un contexte donné. Le **Conditional Access** décide, à chaque connexion, s'il faut autoriser, renforcer ou bloquer, à partir des signaux disponibles.

## Méthodes d'authentification

La configuration se fait dans Protection puis Authentication methods puis Policies. Pour chaque méthode, on active ou désactive, on cible des utilisateurs ou des groupes, on exclut, et on règle des options propres à la méthode.

| Méthode | Résiste au phishing | Remarque |
|---|---|---|
| Passkey FIDO2 | Oui | Clé matérielle ou passkey dans Microsoft Authenticator |
| Windows Hello Entreprise | Oui | Lié à l'appareil |
| Certificate-based authentication | Oui | Authentification directe par certificat X.509, sans fédération |
| Microsoft Authenticator | Non | Number matching et contexte imposés depuis mai 2023 |
| Temporary Access Pass | Non | Credential d'amorçage, durée limitée |
| OATH hardware et software token | Non | Code à 6 chiffres, 30 secondes |
| SMS et appel vocal | Non | À éviter, interceptable |
| Mot de passe | Non | Ne se désactive pas, se contourne |

Deux points d'actualité que l'examen teste depuis la révision d'avril 2026 :

La gestion des méthodes a été **centralisée**. Les anciens écrans « legacy MFA settings » et l'onglet de méthodes de la policy SSPR ne servent plus à gérer les méthodes. Le portail expose un contrôle **Manage migration** avec trois états : Pre-migration, Migration in progress, Migration complete. Tant que la migration n'est pas terminée, les deux configurations coexistent et la policy moderne l'emporte sur les méthodes qu'elle gère explicitement.

Une méthode activée pour un utilisateur ne signifie pas qu'il l'a enregistrée. L'écart entre le périmètre autorisé et le périmètre réellement enregistré se lit dans Authentication methods puis **Registration and reset activity** et dans le rapport **User registration details**. C'est là qu'on identifie les utilisateurs qui ne pourront pas satisfaire une future exigence de MFA résistante au phishing.

### Temporary Access Pass

Un TAP est un code temporaire à durée de vie limitée qui permet de s'authentifier sans mot de passe et sans méthode déjà enregistrée. Il sert à l'onboarding, à l'amorçage d'une méthode passwordless, et à la récupération quand l'utilisateur a perdu son téléphone ou sa clé.

Ce que l'examen vérifie :

Il est créé **par un administrateur pour un autre utilisateur**, jamais pour soi-même. Le rôle requis est Authentication Administrator pour les comptes non-administrateurs, ou Privileged Authentication Administrator pour agir aussi sur les comptes administrateurs. Global Administrator convient également.

Le caractère à usage unique se décide **à la création de chaque pass**, via `isUsableOnce`. La policy ne fait que plafonner ce choix : tant que son réglage One-time use reste à False, l'administrateur peut créer indifféremment un pass à usage unique ou multiple ; dès que la policy passe à True, tous les pass deviennent obligatoirement à usage unique.

La policy fixe aussi la durée de vie minimale et maximale (de 10 minutes à 30 jours, 1 heure par défaut) et le délai avant activation.

Un TAP satisfait l'exigence de MFA, ce qui permet à son porteur d'enregistrer une méthode dans un tenant qui exige la MFA pour l'enregistrement. En revanche il ne satisfait aucune authentication strength résistante au phishing, et un utilisateur ne peut pas s'en servir pour changer son mot de passe via le self-service password reset.

### Self-service password reset et protection des mots de passe

Le SSPR se règle dans Protection puis Password reset. On y définit le périmètre (personne, un groupe, tout le monde), le nombre de méthodes requises pour la réinitialisation, et l'exigence ou non de réenregistrement périodique. En environnement hybride, le password writeback est indispensable pour que la réinitialisation redescende dans l'Active Directory local.

Microsoft Entra Password Protection bloque les mots de passe faibles à partir d'une liste globale gérée par Microsoft et d'une **liste personnalisée** d'au plus 1 000 termes propres à l'organisation. Déployée en local via un agent sur les contrôleurs de domaine, elle applique les mêmes règles aux changements effectués dans l'Active Directory. Le mode audit permet de mesurer l'impact avant application. La fonctionnalité exige Microsoft Entra ID P1.

### Authentication strength

Une authentication strength est un ensemble nommé de combinaisons de méthodes considérées comme suffisantes. Microsoft en fournit trois intégrées : Multifactor authentication, Passwordless MFA, et Phishing-resistant MFA. On peut en créer des personnalisées.

La distinction à tenir : la policy de méthodes définit ce qui est **disponible** dans le tenant, l'authentication strength définit ce qui est **acceptable** pour un accès donné. Une strength qui exige une méthode non activée dans la policy rend l'accès impossible.

Dans une policy Conditional Access, « Require authentication strength » et « Require multifactor authentication » sont **mutuellement exclusifs** : le portail interdit de cocher les deux dans la même policy.

## Conditional Access

C'est le moteur de décision. À chaque tentative d'accès, il évalue les policies applicables et rend une décision.

### Structure réelle d'une policy

Le découpage du portail est le suivant, et le confondre coûte des points.

**Assignments** contient quatre blocs : Users, Target resources, Network et Conditions.

- *Users* cible des utilisateurs, des groupes, des rôles d'annuaire, des invités et utilisateurs externes par type, ou des workload identities.
- *Target resources* cible des applications cloud, des actions utilisateur (enregistrement d'une méthode, jonction d'appareil), ou un **authentication context**.
- *Network* cible des emplacements réseau. Il a été sorti du bloc Conditions et vit désormais à part, ce qui permet de le combiner avec Global Secure Access.
- *Conditions* regroupe le reste : risque utilisateur, risque de connexion, risque interne, plateforme d'appareil, applications clientes, état de l'appareil, flux d'authentification.

**Access controls** contient deux blocs : Grant et Session.

*Grant* décide d'accorder ou de bloquer, et sous quelle condition : MFA, appareil marqué conforme, appareil hybrid joined, application cliente approuvée, application protégée par une app protection policy, changement de mot de passe, ou authentication strength. Quand plusieurs contrôles sont cochés, on choisit « Require all » ou « Require one ».

*Session* agit après l'octroi : sign-in frequency, persistent browser session, **continuous access evaluation**, Conditional Access App Control (routage vers Defender for Cloud Apps), restrictions imposées par l'application, protection du token, et désactivation des valeurs par défaut de résilience.

La MFA est donc un **grant control**, jamais une condition. Une condition décrit le contexte, un grant control décrit l'exigence.

### Continuous access evaluation

Sans CAE, un access token reste valide jusqu'à son expiration, environ une heure, même si l'utilisateur est désactivé entre-temps. La CAE fait dialoguer Entra et les ressources compatibles (Exchange Online, SharePoint Online, Teams, Microsoft Graph) pour révoquer une session quasi immédiatement sur des événements critiques : compte désactivé ou supprimé, mot de passe modifié, révocation explicite des tokens, risque utilisateur élevé détecté, ou changement d'emplacement réseau quand la policy le prévoit.

Elle est activée par défaut. Le réglage dans la policy CA sert principalement à la désactiver pour dépannage, ou à activer le **strict location enforcement**, qui rejette un token présenté depuis une adresse IP hors des emplacements autorisés.

### Authentication context et protected actions

Un **authentication context** permet d'appliquer une policy CA à une opération précise plutôt qu'à une application entière. On définit un contexte (par exemple `c1`, « Données financières »), on l'assigne comme cible d'une policy CA exigeant par exemple une MFA résistante au phishing, puis une application, un site SharePoint étiqueté, ou une opération Entra s'y rattache. Le reste de l'application reste soumis aux règles normales.

Les **protected actions** appliquent ce mécanisme aux opérations d'administration d'Entra elles-mêmes. On lie un authentication context à des permissions précises, par exemple la mise à jour des policies Conditional Access ou la suppression d'un rôle, de sorte qu'un administrateur déjà connecté doive satisfaire une exigence renforcée avant d'exécuter cette opération. C'est la réponse attendue quand un scénario demande de protéger la modification des policies CA sans exiger davantage pour le reste de l'administration.

### Déploiement et validation

**Report-only** évalue la policy et journalise le résultat sans l'appliquer. C'est le passage obligé avant toute mise en production. Les résultats se lisent dans l'onglet Conditional Access d'un log de connexion, et dans le classeur Conditional Access Insights and Reporting.

**What If** simule le résultat d'un ensemble de signaux fournis à la main, sans connexion réelle.

Les **templates** fournissent seize policies préconfigurées couvrant les scénarios recommandés (MFA pour les administrateurs, blocage de l'authentification héritée, exigence d'appareil conforme). Ils se créent directement en report-only.

Enfin, une policy CA ne peut coexister avec les **Security Defaults**. Ce n'est pas une nuance de vocabulaire mais une contrainte technique : tant que les Security Defaults sont activés, la création d'une policy CA est refusée, et il faut les désactiver dans Microsoft Entra ID puis Properties puis Manage security defaults pour basculer. Les Security Defaults sont gratuits et imposent la MFA à tous ; le Conditional Access exige P1 et permet le ciblage.

## Microsoft Entra ID Protection

Le nom officiel est bien Microsoft Entra ID Protection. Le produit calcule et expose du risque.

**Le risque utilisateur** évalue la probabilité que le compte lui-même soit compromis, à partir de signaux persistants comme des identifiants retrouvés dans une fuite ou une activité anormale cumulée.

**Le risque de connexion** évalue la probabilité qu'une tentative précise ne soit pas légitime : voyage impossible, adresse IP anonyme, propriétés de connexion inhabituelles, adresse liée à un logiciel malveillant.

**Les risk detections** sont les événements individuels qui alimentent ces deux scores. Certaines sont calculées en temps réel et peuvent bloquer la connexion en cours, d'autres hors ligne et n'apparaissent qu'après coup.

Trois rapports servent au suivi : Risky users, Risky sign-ins et Risk detections. Un administrateur peut confirmer une compromission, ce qui force le risque à High et alimente l'apprentissage, ou rejeter le risque, ce qui le remet à zéro.

Les licences sont le principal discriminant sur ce sujet :

| Capacité | Licence |
|---|---|
| Voir que des utilisateurs et connexions sont à risque, sans détail | Free |
| Rapports détaillés, niveau de risque, remédiation en libre-service | P2 |
| Policies de risque utilisateur et de risque de connexion, y compris portées par CA | P2 |
| Exporter les risk detections vers Log Analytics ou Sentinel | P1 pour l'export, P2 pour le contenu |

La bonne pratique actuelle est de porter les exigences de risque dans des policies Conditional Access plutôt que dans les policies héritées d'ID Protection, ce qui permet de combiner le risque avec les autres signaux.

## Defender for Cloud Apps

Defender for Cloud Apps apporte la visibilité sur l'usage des applications cloud et le contrôle en temps réel des sessions.

Une **access policy** décide si l'accès à l'application est autorisé ou bloqué. Une **session policy** contrôle ce que l'utilisateur peut faire pendant la session : bloquer un téléchargement, appliquer une étiquette de confidentialité à un fichier téléchargé, bloquer un copier-coller ou une impression, surveiller sans bloquer.

Le prérequis vaut pour les deux et il est régulièrement testé : l'application doit être routée vers le reverse proxy de Defender for Cloud Apps par une policy **Conditional Access** dont le session control est **Conditional Access App Control**. Sans ce routage, aucune des deux familles de policies ne s'applique. Conditional Access App Control n'est donc pas réservé aux session policies : c'est le canal commun.

## Global Secure Access

Global Secure Access est le volet Security Service Edge de Microsoft Entra. Il est apparu dans les compétences mesurées et comporte quatre volets.

Le **client Global Secure Access** s'installe sur Windows, macOS, iOS et Android et capture le trafic à destination des profils de transfert configurés. Un déploiement sans client est possible pour des réseaux distants via un tunnel IPsec depuis l'équipement de bordure.

**Microsoft Entra Private Access** publie des applications privées, sur site ou dans un cloud privé, sans exposer le réseau. Le trafic passe par un **private network connector** installé à proximité de la ressource, qui n'ouvre que des connexions sortantes. Contrairement à un VPN, l'accès est accordé application par application et non au réseau entier, et chaque application publiée peut porter ses propres policies Conditional Access. C'est le successeur fonctionnel du proxy d'application, dont il partage le connecteur, avec deux différences majeures : Private Access couvre tous les protocoles TCP et UDP et pas seulement HTTP et HTTPS, et il ne publie pas d'URL sur Internet.

**Microsoft Entra Internet Access** sécurise le trafic sortant vers Internet, avec du filtrage web par catégorie et par nom de domaine, et applique le Conditional Access au trafic réseau lui-même.

**Microsoft Entra Internet Access for Microsoft 365** traite spécifiquement le trafic Microsoft 365 et permet le **compliant network check**, un contrôle Conditional Access qui vérifie que la connexion transite bien par le réseau Microsoft, ce qui bloque le vol de token présenté depuis ailleurs.

Les emplacements réseau de Global Secure Access s'utilisent directement dans le bloc Network d'une policy Conditional Access.

## Ce qui se joue sur des détails

La MFA est un grant control, pas une condition, et elle ne peut pas être cochée en même temps qu'une authentication strength.

L'emplacement réseau n'est plus une condition dans le portail actuel : c'est un bloc à part dans Assignments.

Les Security Defaults et le Conditional Access s'excluent techniquement, ce n'est pas une simple recommandation.

Un TAP est créé par un administrateur pour quelqu'un d'autre, et son caractère à usage unique se décide pass par pass, pas seulement dans la policy.

Une session policy Defender for Cloud Apps ne fonctionne que si une policy Conditional Access route l'application via Conditional Access App Control.

Les policies de risque exigent P2. Un énoncé qui mentionne un tenant en P1 élimine toute réponse fondée sur le risque utilisateur ou le risque de connexion.

Le résultat « MFA requirement satisfied by claim in the token » dans un log signifie que l'exigence a été satisfaite par un claim déjà présent : aucune nouvelle MFA n'a été demandée à l'utilisateur pour cet événement.
