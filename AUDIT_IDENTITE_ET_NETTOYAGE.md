# Audit d’identité et de nettoyage — Conduit

**Dépôt audité :** `Frankenstein-Labs/conduit`  
**Parent GitHub :** `cogwheel0/conduit`  
**Version observée :** `4.1.7`  
**Date de l’audit :** 24 septembre 2026  
**Périmètre :** audit non destructif ; aucun fichier fonctionnel n’a été supprimé ou modifié.

## Conclusion exécutive

Le dépôt n’est pas encombré par un ancien produit sans rapport. Il s’agit d’un fork encore très proche de Conduit amont. Les éléments à remplacer sont principalement l’identité de l’application, les identifiants de bundle/package, les liens de distribution, les coordonnées du mainteneur, les informations de licence et certains noms de classes générées. Les intégrations Open WebUI, Hermes, OpenRouter, Ollama et les fournisseurs compatibles OpenAI ne sont pas des restes à supprimer : elles constituent la fonctionnalité principale à préserver.

Il faut donc éviter une suppression globale de chaînes telles que `Conduit`, `Open WebUI`, `Hermes` ou `OpenRouter`. La bonne démarche est une re-marque contrôlée, suivie d’une régénération des bindings et d’une vérification complète Android/iOS.

## 1. Éléments hérités à remplacer

### 1.1 Identité publique et branding

Les éléments suivants portent encore l’identité amont :

| Zone | Emplacement | Constat | Action future |
|---|---|---|---|
| Nom public | `README.md`, `android/app/src/main/AndroidManifest.xml`, `ios/Runner/Info.plist`, traductions et ressources | Nom affiché : `Conduit` | Remplacer par le nom de l’entreprise ou du produit |
| Icônes et splash screens | `assets/icons/`, ressources Android et `ios/Runner/Assets.xcassets/` | Assets Conduit et Open WebUI présents | Remplacer les assets propriétaires ou amont selon les droits |
| Package Android | `android/app/build.gradle.kts` lignes 17 et 22 | `app.cogwheel.conduit` | Choisir un nouvel identifiant permanent, par exemple `com.<entreprise>.<produit>` |
| Bundle iOS | `ios/Runner.xcodeproj/project.pbxproj` et configurations des extensions | `app.cogwheel.conduit.*` | Choisir le bundle ID principal et ceux des widgets/extensions |
| URL schemes iOS | `ios/Runner/Info.plist` | Schemes et noms contenant `app.cogwheel.conduit` | Régénérer après choix du nouvel identifiant |
| Background task iOS | `ios/Runner/Info.plist` | `app.cogwheel.conduit.refresh` | Renommer avec le nouvel identifiant |
| Namespace Android/Pigeon | `pigeons/conduit_platform_apis.dart`, code Kotlin généré | Package `app.cogwheel.conduit` | Modifier la source Pigeon puis régénérer, ne pas éditer uniquement les fichiers générés |
| Noms de classes natives | Android et iOS | Classes telles que `ConduitApplication`, `ConduitWidgetProvider`, `ConduitPlatformApis` | Décider si un renommage complet est souhaité ; ce n’est pas nécessaire pour changer le nom visible |

### 1.2 Références au mainteneur et à l’amont

Les références suivantes doivent être revues pour une distribution sous ton entreprise :

- `README.md` : badges, liens vers les stores amont, liens GitHub amont, sponsors et contact `cogwheel@cogwheel.app` ;
- `README.md` : mention de l’offre enterprise/white-label de l’ancien mainteneur ;
- `README.md` : mention « independent client » et relation avec Open WebUI ;
- `PRIVACY_POLICY.md` : date, contact, nom du produit et description de la responsabilité du fournisseur ;
- `LICENSE` : copyright `2026 Tunap Paul` ; cette ligne ne doit pas être supprimée sans analyse de licence ;
- `.github/FUNDING.yml` : financement de l’amont ; à remplacer ou retirer selon la stratégie de l’entreprise ;
- scripts et workflows de release : références aux conventions de publication amont et aux secrets de signature.

### 1.3 Références techniques au nom Conduit

Le nom `Conduit` apparaît également dans des symboles techniques : widgets, services, thèmes, classes Swift/Kotlin, routes et bindings Pigeon. Il ne faut pas les remplacer mécaniquement avant d’avoir décidé si l’objectif est :

1. un simple rebranding visuel ; ou
2. un fork entièrement renommé au niveau du code et des namespaces.

Le second choix est beaucoup plus coûteux et touche les fichiers générés. Toute modification de Pigeon doit être faite dans `pigeons/conduit_platform_apis.dart`, puis régénérée avec l’outil dédié décrit dans `docs/BUILDING.md`.

## 2. Éléments fonctionnels à conserver

Les composants suivants ne sont pas des traces d’un ancien projet à effacer :

- `lib/features/auth/` : connexion à des serveurs Open WebUI, mot de passe, LDAP, SSO/OAuth et proxy auth ;
- `lib/core/auth/` : stockage sécurisé des identifiants, tokens, cookies, headers et intercepteurs réseau ;
- `lib/features/direct_connections/` : profils OpenAI-compatible, Ollama et OpenRouter ;
- `lib/features/hermes/` : transport et interface pour l’agent Hermes ;
- `openwebui-src/` et `hermes-src/` : sous-modules de référence des contrats API, non embarqués comme backend de l’application ;
- `third_party/mermaid/` et `third_party/katex/` : dépendances nécessaires au rendu natif ;
- la gestion `flutter_secure_storage` : elle est essentielle si des clés OpenRouter sont saisies dans l’application ;
- les fonctionnalités de streaming, pièces jointes, voix, notes, terminal et synchronisation.

La documentation indique aussi qu’OpenRouter est déjà pris en charge comme connexion directe. Il ne faudra donc probablement pas « ajouter OpenRouter » depuis zéro : il faudra plutôt configurer, tester et éventuellement améliorer son parcours utilisateur.

## 3. Authentification : constat important

Le code actuel ne montre pas de compte propriétaire de l’application avec une base d’utilisateurs indépendante. Il n’y a pas, dans `pubspec.yaml`, de couche évidente de type Firebase Auth, Supabase Auth, Auth0 ou backend d’identité maison.

L’authentification existante concerne principalement :

- le compte utilisateur sur son serveur Open WebUI ;
- les fournisseurs OAuth/SSO exposés par ce serveur ;
- les credentials de connexions directes, stockés sur l’appareil ;
- OAuth pour certains serveurs MCP ou services connectés.

En conséquence, « ajouter Google et GitHub » peut vouloir dire deux choses différentes :

- **OAuth de l’infrastructure Open WebUI du client** : le serveur doit être configuré pour proposer ces fournisseurs ; l’application doit seulement gérer correctement le flux retourné ;
- **compte propriétaire de ton application** : il faut une identité indépendante, un backend ou un fournisseur d’identité, des sessions, une politique de confidentialité, la suppression de compte et une stratégie de stockage des données.

Ce choix ne doit pas être implémenté par simple ajout de boutons dans l’écran de connexion.

## 4. OpenRouter et gestion des clés

Le code comporte déjà une implémentation OpenRouter dans `lib/features/direct_connections/services/openai_compatible_adapter.dart` et dans les modèles/écrans de connexions directes.

Pour la première version :

- l’utilisateur peut saisir sa propre clé OpenRouter ;
- la clé doit rester dans `flutter_secure_storage` / Keychain / Keystore ;
- la clé ne doit pas être placée dans le dépôt, dans une constante Dart, dans le bundle ou dans le code d’un backend public ;
- le test de connexion doit vérifier le catalogue de modèles et une génération minimale sans exposer la clé dans les logs.

Pour une offre commerciale à grande échelle, une clé OpenRouter d’entreprise ne doit pas être embarquée dans l’application mobile. Il faudrait alors un backend de proxy, une gestion de quotas, une facturation et une protection contre l’extraction de la clé.

## 5. Points de licence et distribution

Le projet est sous **GPL-3.0**. La GPL permet l’utilisation commerciale, mais la distribution d’une version modifiée entraîne des obligations de licence, de notices et de mise à disposition du code source correspondant selon le mode de distribution. Les sous-modules et les assets tiers peuvent avoir leurs propres obligations.

À examiner avant publication :

- conservation des notices de copyright ;
- `THIRD_PARTY_NOTICES.md` ;
- licences des dépendances Flutter et des sous-modules ;
- obligations liées à la distribution iOS et Android ;
- politique de confidentialité, conditions d’utilisation et traitement des données ;
- compatibilité entre une offre white-label et la GPL-3.0.

Cet audit n’est pas un avis juridique ; une validation par un conseil spécialisé open source est recommandée avant commercialisation.

## 6. Éléments à ne pas supprimer par erreur

Les suppressions suivantes seraient dangereuses sans remplacement fonctionnel :

- `openwebui-src/` et `hermes-src/` ;
- les adaptateurs et parseurs de streaming ;
- `SecureCredentialStorage` et les tests associés ;
- les fichiers Pigeon et bindings natifs ;
- les extensions iOS de partage/widgets ;
- les permissions audio, caméra, réseau local et fichiers si les fonctions correspondantes restent activées ;
- les notices tierces et la licence GPL ;
- les fichiers de localisation `lib/l10n/*.arb`.

## 7. Plan de nettoyage recommandé, sans exécution dans cet audit

### Phase 1 — Décision d’identité

Définir le nom public, le domaine, le bundle ID Android, le bundle ID iOS, les identifiants des extensions, l’icône, les couleurs et les coordonnées de support.

### Phase 2 — Rebranding contrôlé

Remplacer les métadonnées publiques, les identifiants de package et les assets. Conserver d’abord les noms internes `Conduit` si cela réduit le risque, puis décider plus tard si un renommage technique complet apporte une réelle valeur.

### Phase 3 — Authentification

Décider si l’application reste un client sans compte propriétaire, si elle ajoute un compte propriétaire Google/GitHub, ou si elle supporte les deux. Cette décision détermine le backend, les redirections OAuth et le modèle de données.

### Phase 4 — OpenRouter

Utiliser la connexion directe déjà présente pour une clé personnelle. Ne jamais committer de clé. Ajouter ensuite des tests de non-fuite, des messages d’erreur propres et éventuellement une gestion de quotas si l’entreprise fournit ses propres credentials.

### Phase 5 — Validation

Après chaque groupe de changements :

```bash
flutter pub get
dart run build_runner build --delete-conflicting-outputs
flutter analyze
flutter test
```

Puis vérifier séparément les builds Android, iOS, les widgets, le partage, les deep links et les parcours OAuth sur appareils réels.

## État au terme de l’audit

- **Fichiers supprimés :** aucun.
- **Fichiers modifiés :** aucun fichier fonctionnel ; ce rapport uniquement.
- **Sous-modules :** initialisés et disponibles.
- **Prochaine décision nécessaire :** nom/identité de l’application et périmètre exact de l’authentification Google/GitHub.
