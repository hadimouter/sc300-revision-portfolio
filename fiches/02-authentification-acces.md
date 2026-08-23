# Authentification et gestion des accès

Domaine « Implement authentication and access management », 25 à 30 % de l'examen. C'est le domaine le plus lourd en scénarios.

Le modèle mental tient en trois couches :

1. **Authentication methods policy** : quelles méthodes les utilisateurs ont le droit d'utiliser ;
2. **Authentication strength** : quelles combinaisons sont suffisamment fortes pour un accès donné ;
3. **Conditional Access** : dans quel contexte autoriser, renforcer ou bloquer l'accès.

## Méthodes d'authentification

| Méthode | Phishing-resistant ? | À retenir |
|---|---|---|
| Passkey / FIDO2 | Oui | clé matérielle ou passkey prise en charge |
| Windows Hello for Business | Oui | clé liée à l'appareil, pas simple mot de passe local |
| Certificate-based authentication | Oui | certificat X.509 directement avec Entra |
| Microsoft Authenticator | Pas toujours | MFA / passwordless selon mode |
| Temporary Access Pass | Non | credential temporaire d'amorçage / recovery |
| OATH | Non | OTP |
| SMS / voice | Non | méthode faible |
| Password | Non | facteur de connaissance |

Autoriser une méthode ne signifie pas que l'utilisateur l'a enregistrée. Les rapports d'enregistrement servent à mesurer le décalage entre méthode **autorisée** et méthode **réellement enregistrée**.

### Temporary Access Pass

Un TAP est créé par un administrateur pour un autre utilisateur afin d'amorcer une méthode passwordless ou de récupérer l'accès.

À retenir :

- Authentication Administrator pour les utilisateurs standards ;
- Privileged Authentication Administrator pour agir sur certains comptes administrateurs ;
- durée limitée ;
- configuration one-time / multi-use selon la policy et le pass ;
- peut satisfaire une exigence MFA, mais n'est pas pour autant une méthode phishing-resistant.

### Certificate-Based Authentication et high-affinity binding

CBA permet de s'authentifier directement avec un certificat X.509.

L'examen peut tester le **binding** entre certificat et utilisateur. Pour un binding à forte affinité, reconnaître notamment :

> **X509SKI / Subject Key Identifier (SKI)**

Ce type d'identifiant lie plus fortement le certificat / la clé à l'identité qu'une correspondance générique basée uniquement sur un nom.

### Windows Hello for Business

Windows Hello for Business utilise une paire de clés liée à l'appareil. Le PIN ou la biométrie sert localement à déverrouiller la clé ; ce n'est pas le secret transmis à distance comme un mot de passe.

En environnement hybride, **Cloud Kerberos Trust** permet à un utilisateur Windows Hello for Business d'accéder à des ressources AD DS protégées par Kerberos sans déployer le modèle plus lourd de certificate trust.

À reconnaître :

```text
Windows Hello for Business + accès Kerberos on-prem
→ Cloud Kerberos Trust
→ Microsoft Entra Kerberos
```

### Registration campaigns

Les registration campaigns poussent les utilisateurs ciblés à enregistrer une méthode moderne. Elles ne remplacent pas la methods policy : l'une autorise / cible, l'autre aide à faire réellement enregistrer la méthode.

## SSPR et Password Protection

Self-service password reset permet aux utilisateurs autorisés de réinitialiser leur mot de passe selon les méthodes/configurations disponibles.

En environnement hybride, **password writeback** permet de répercuter le changement vers AD DS dans les scénarios pris en charge.

Microsoft Entra Password Protection combine une liste globale gérée par Microsoft et une liste personnalisée. Avec les composants on-premises appropriés, les contrôles peuvent aussi protéger les changements de mot de passe AD DS.

## Authentication strength

Une authentication strength décrit les combinaisons de méthodes jugées suffisantes.

Raccourci :

```text
Methods policy
= ce que l'utilisateur peut utiliser

Authentication strength
= ce qui est accepté pour CET accès
```

Exemple : une application très sensible peut exiger une strength **Phishing-resistant MFA**.

## Désactiver un compte et révoquer des sessions

Désactiver un compte empêche les nouvelles authentifications, mais il faut distinguer cela de la validité de tokens déjà émis.

Pour révoquer les sessions utilisateur avec Microsoft Graph PowerShell, la commande à reconnaître est :

```powershell
Revoke-MgUserSignInSession -UserId <id>
```

La révocation des sessions agit sur la capacité à renouveler / continuer les sessions selon le mécanisme. Elle ne doit pas être décrite comme une suppression magique de tous les JWT déjà remis.

Avec **Continuous Access Evaluation**, certaines ressources et certains clients compatibles peuvent réagir à des événements critiques, comme la désactivation d'un compte, et rejeter un access token avant son expiration normale.

## Conditional Access

Conditional Access est le moteur de décision contextuel.

### Structure d'une policy

**Assignments** :

- Users / workload identities selon scénario ;
- Target resources ;
- Network ;
- Conditions comme device platform, client apps, risk, device state, authentication flows.

**Access controls** :

- Grant ;
- Session.

### Grant controls

Ils déterminent ce qui doit être satisfait pour obtenir l'accès, par exemple :

- require MFA ;
- require authentication strength ;
- require compliant device ;
- require hybrid joined device ;
- block access.

La MFA est donc un **grant control**, pas une condition.

### Session controls

Ils agissent après l'octroi :

- sign-in frequency ;
- persistent browser session ;
- Conditional Access App Control ;
- contrôles de token / session pris en charge ;
- comportements liés à CAE selon configuration.

### Report-only et What If

**Report-only** évalue la policy sans la faire bloquer réellement. Les résultats sont visibles dans les logs et les outils de reporting.

**What If** simule quelles policies s'appliqueraient à un scénario donné sans imposer une connexion réelle.

### Templates

Les templates accélèrent le déploiement de policies recommandées. Ils ne dispensent pas de vérifier le ciblage, les exclusions et l'impact avant activation.

### Device-enforced restrictions

Le study guide nomme explicitement les restrictions appliquées par l'application selon le contexte de l'appareil.

Ne pas confondre :

- **require compliant device** = condition d'octroi ;
- **application-enforced/device-enforced restriction** = accès éventuellement accordé mais expérience limitée sur l'appareil ;
- **Defender for Cloud Apps session policy** = contrôle via reverse proxy pendant la session.

### Continuous Access Evaluation

CAE permet à Entra et aux ressources compatibles de réagir à certains événements sans attendre uniquement l'expiration normale du token.

Exemples d'événements critiques selon prise en charge :

- compte désactivé ou supprimé ;
- changement / réinitialisation de mot de passe selon scénario ;
- révocation de sessions ;
- certains changements de risque / localisation selon ressource et policy.

Important : **CAE n'est pas un mécanisme universel pour tous les tokens et toutes les ressources**, et les critical events CAE ne doivent pas être résumés au vieux raccourci « nécessite forcément P1 ».

### Authentication context

Permet d'appliquer une exigence CA à une opération ou zone sensible précise plutôt qu'à toute l'application.

### Protected actions

Appliquent un authentication context à certaines opérations d'administration Entra, par exemple protéger la modification de policies sensibles par une authentification renforcée au moment précis de l'action.

### Security Defaults vs Conditional Access

Security Defaults fournit une protection globale simple. Conditional Access fournit du ciblage et des contrôles détaillés.

Dans un tenant qui veut gérer ses propres policies CA, il faut traiter explicitement Security Defaults au lieu de supposer que les deux mécanismes forment deux couches indépendantes et configurables en parallèle.

## Microsoft Entra ID Protection

### User risk

Probabilité que **l'identité elle-même** soit compromise.

### Sign-in risk

Probabilité qu'**une tentative de connexion précise** soit illégitime.

### Risk detections

Événements individuels qui alimentent le risque, avec calcul en temps réel ou hors ligne selon le type.

Les trois vues à reconnaître :

- Risky users ;
- Risky sign-ins ;
- Risk detections.

Les policies avancées basées sur user risk / sign-in risk et la remédiation détaillée relèvent de Microsoft Entra ID P2.

## Defender for Cloud Apps

Defender for Cloud Apps apporte découverte, classification et contrôle des applications cloud.

### Cloud Discovery

Identifie l'usage réel des applications cloud et le **Shadow IT**.

### Cloud App Catalog

Catalogue les applications cloud et fournit des informations / scores de risque.

### Connected apps

Connecte Defender for Cloud Apps à des services SaaS pris en charge pour obtenir de la visibilité et appliquer certaines fonctions de gouvernance.

### Application-enforced restrictions

Certaines applications peuvent appliquer des restrictions selon les signaux Conditional Access et l'état de l'appareil.

### Conditional Access App Control

Route la session vers le reverse proxy de Defender for Cloud Apps pour contrôle en temps réel.

- **Access policy** : autoriser ou bloquer l'entrée ;
- **Session policy** : contrôler les actions pendant la session, par exemple téléchargement, copie ou impression.

### OAuth app policies

Gouvernent les applications OAuth connectées, notamment selon permissions et niveau de risque.

Ne pas confondre avec Conditional Access, qui décide de l'accès d'une identité à une ressource.

## Global Secure Access

Global Secure Access regroupe notamment Microsoft Entra Private Access et Internet Access.

### Global Secure Access client

Le client capture le trafic concerné selon les profils de forwarding sur les plateformes prises en charge.

### Microsoft Entra Private Access

Publie des applications privées sans exposer directement le réseau. Le **private network connector** se place près de la ressource et établit des connexions sortantes.

Différence avec VPN : accès **application par application** au lieu de donner une connectivité réseau large.

### Microsoft Entra Internet Access

Protège / contrôle le trafic sortant Internet selon les fonctionnalités et licences activées.

### Microsoft 365 traffic

Des fonctions dédiées au trafic Microsoft 365 permettent d'intégrer les signaux réseau à Conditional Access, par exemple des contrôles vérifiant que le trafic transite par le chemin attendu.

## Ce qui se joue sur des détails

- Methods policy = méthode disponible ; authentication strength = méthode acceptable.
- CBA high-affinity -> **X509SKI / SKI**.
- WHfB hybride + Kerberos -> **Cloud Kerberos Trust**.
- Révoquer sessions utilisateur -> `Revoke-MgUserSignInSession`.
- MFA = grant control.
- Report-only != What If.
- Device-enforced restriction != require compliant device.
- CAE peut rejeter un token avant `exp` sur scénarios compatibles, mais n'efface pas tous les tokens universellement.
- User risk = compte ; sign-in risk = tentative.
- Cloud Discovery = Shadow IT.
- Cloud App Catalog = score / informations sur applications cloud.
- OAuth app policy = gouvernance des apps OAuth.
- Access policy = entrer ou non ; session policy = comportement pendant la session.
- Private Access = application privée + private network connector, pas VPN réseau complet.
- `MFA requirement satisfied by claim in the token` signifie qu'aucune nouvelle MFA n'a nécessairement été demandée pour cet événement.
