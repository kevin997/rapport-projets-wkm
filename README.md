# RAPPORT DE CONTRE-AUDIT TECHNIQUE — Projets ABBEAV & TEATCHA

**Date :** 28 Février 2026  
**Auditeur :** Audit Indépendant de OVANGA LIBOIRE KEVIN, Dev FullStack de www.csl-brands.com (analyse du code source)  
**Objectif :** Valider ou infirmer le rapport technique de TERI Armand (25/02/2026)

---

## 1. RÉSUMÉ EXÉCUTIF

L'analyse indépendante du code source confirme **en grande partie** le rapport du développeur sur le projet ABBEAV. L'architecture est solide et professionnelle. Certains points méritent néanmoins des **nuances** et des **clarifications**.

> [!IMPORTANT]
> **Verdict global :** Le développeur a produit un travail sérieux, structuré et professionnel sur ABBEAV. L'architecture 3 couches (Service → Repository → Providers) est connectée à Supabase avec un système de cache sophistiqué. Le projet TEATCHA est moins avancé, avec des données encore en dur.

---

## 2. INVENTAIRE DES PROJETS TROUVÉS

| Projet           | Technologie                   | Fichiers Source      | Commits Git        | Description                         |
| ---------------- | ----------------------------- | -------------------- | ------------------ | ----------------------------------- |
| `abbeav`         | Flutter + Supabase + Riverpod | **80 fichiers Dart** | N/A (accès refusé) | Version principale sur App Store    |
| `abbeav-v1`      | Flutter + Supabase + Provider | **40 fichiers Dart** | **1 seul commit**  | Version d'extension/codes à activer |
| `teatcha`        | Flutter + Firebase + Provider | **47 fichiers Dart** | N/A (accès refusé) | App éducative                       |
| `abbeav-support` | React + Vite + TailwindCSS    | **66 fichiers TSX**  | **1 seul commit**  | Page web de support (landing)       |

---

## 3. VALIDATION POINT PAR POINT

### 3.1 Architecture Technique

| Affirmation du rapport            | ✅/❌                   | Constat réel                                                                                                                                                                                                                                                                                                                           |
| --------------------------------- | ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Flutter (Dart) cross-platform     | ✅ **VRAI**             | Confirmé - SDK ^3.8.1, configs iOS + Android présentes                                                                                                                                                                                                                                                                                 |
| Supabase comme BaaS (ABBEAV)      | ✅ **VRAI**             | Confirmé - `supabase_flutter: ^2.12.0`, .env avec URL et clé anon                                                                                                                                                                                                                                                                      |
| PostgreSQL via Supabase           | ✅ **VRAI**             | Fichiers SQL ([commands.sql](file:///home/atlas/Projects/WMK/abbeav/commands.sql), [insert.sql](file:///home/atlas/Projects/WMK/abbeav/insert.sql)) présents + [media_service.dart](file:///home/atlas/Projects/WMK/abbeav/lib/modules/media/data/services/media_service.dart) (1779 lignes) fait des requêtes PostgreSQL via Supabase |
| Supabase Auth (passwordless)      | ⚠️ **PARTIELLEMENT**    | Auth codée mais **auth guard commenté** dans le router. Le système d'auth n'est pas actif en mode dev                                                                                                                                                                                                                                  |
| Supabase Storage + CDN            | ✅ **VRAI**             | URLs Supabase S3 signées confirmées dans la base de données                                                                                                                                                                                                                                                                            |
| Firebase pour TEATCHA             | ✅ **VRAI**             | `firebase_core`, `cloud_firestore`, `firebase_auth` dans les dépendances                                                                                                                                                                                                                                                               |
| Architecture Serverless Full BaaS | ✅ **VRAI pour ABBEAV** | Architecture 3 couches complète : `MediaService` (Supabase) → `MediaRepository` (cache Hive) → Providers (Riverpod)                                                                                                                                                                                                                    |

### 3.1.1 ✅ CORRECTION IMPORTANTE — `save/data.dart` est du CODE MORT

> [!NOTE]
> Lors de l'analyse initiale, le fichier [save/data.dart](file:///home/atlas/Projects/WMK/abbeav/lib/save/data.dart) (2225 lignes) semblait contenir des données hardcodées. **Après vérification approfondie :**
>
> - Le fichier est **ENTIÈREMENT commenté** (encadré par `/*` à la ligne 1 et `*/` à la ligne 2224)
> - Il n'est **importé nulle part** dans le code source actif
> - C'est du **code mort** — probablement un backup historique de la phase de prototypage
>
> **Les données réelles proviennent de Supabase** via :
>
> - [media_service.dart](file:///home/atlas/Projects/WMK/abbeav/lib/modules/media/data/services/media_service.dart) — **1779 lignes** de requêtes Supabase (getHeroCarousel, getBrandBadges, getMediaById, getCast, getStudios, getSimilarMedia, etc.)
> - [media_repository.dart](file:///home/atlas/Projects/WMK/abbeav/lib/modules/media/data/repositories/media_repository.dart) — **1193 lignes** avec stratégie cache-first et stale-while-revalidate via Hive
> - [media_providers.dart](file:///home/atlas/Projects/WMK/abbeav/lib/modules/media/presentation/providers/media_providers.dart) — Providers Riverpod qui utilisent `MediaRepository`

### 3.2 Projet ABBEAV — Fonctionnalités

| Fonctionnalité                | Statut rapport | Constat réel                                                                                                                                                                                                                                                                         | Verdict                                            |
| ----------------------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------- |
| **Inscription/Connexion**     | 50% (Terminé)  | Auth service codé, OTP (pinput), mais **auth guard commenté** dans [router_app.dart](file:///home/atlas/Projects/WMK/abbeav/lib/core/router/router_app.dart) L.60-85. L'app tourne sans authentification en mode dev                                                                 | ⚠️ **Code présent mais désactivé (normal en dev)** |
| **Consultation films/séries** | 100% (Terminé) | Écran `MediaHomeScreen` fonctionnel. Les données proviennent de **Supabase via MediaService** (hero carousel, just release, popular, featured, movies, series, awards, fast). Service de 1779 lignes avec requêtes PostgreSQL                                                        | ✅ **Confirmé — connecté à Supabase**              |
| **Détails films**             | 100% (Terminé) | `MediaDetailScreen` existe, `getMediaById()` fait un JOIN Supabase avec movies, seasons, episodes, cast, studios et similar_media                                                                                                                                                    | ✅ **Confirmé**                                    |
| **Lecture Streaming**         | 100% (Terminé) | `video_player` + `chewie` intégrés. `MediaPlayerScreen` existe. URLs vidéo proviennent de Supabase Storage                                                                                                                                                                           | ✅ **Confirmé**                                    |
| **Fiches Acteurs**            | 100% (Terminé) | `getCast()` et `getCastById()` dans MediaService font des requêtes Supabase sur les tables `media_cast` et `casts`                                                                                                                                                                   | ✅ **Confirmé — connecté à Supabase**              |
| **Fiches Studios**            | 100% (Terminé) | `AboutBrandScreen` existe, `getStudios()` et `getStudioById()` dans MediaService                                                                                                                                                                                                     | ✅ **Confirmé**                                    |
| **Historique visionnage**     | 50% (Terminé)  | Pas de code d'historique visible dans les routes actives                                                                                                                                                                                                                             | ❌ **Non vérifié**                                 |
| **Réservation billets**       | 50% (Terminé)  | **Aucun code** de réservation visible dans le codebase                                                                                                                                                                                                                               | ❌ **Non trouvé dans le code**                     |
| **Votes œuvres**              | 50% (Terminé)  | Pas de système de vote visible dans les fichiers                                                                                                                                                                                                                                     | ❌ **Non trouvé dans le code**                     |
| **Notes et commentaires**     | 50% (Terminé)  | `abbeav-v1` a [reviews_all_service.dart](file:///home/atlas/Projects/WMK/abbeav-v1/lib/services/review/reviews_all_service.dart) et [reviews_all_screen.dart](file:///home/atlas/Projects/WMK/abbeav-v1/lib/view/review/reviews_all_screen.dart) mais pas dans la version principale | ⚠️ **Existe dans -v1 seulement**                   |
| **Gestion de profils**        | 80% (Avancé)   | `ProfileScreen`, `ProfileEditScreen`, `ProfilePasswordScreen` existent                                                                                                                                                                                                               | ✅ **Confirmé**                                    |
| **Playlists et favoris**      | 50% (Terminé)  | `WatchlistScreen` + route existent                                                                                                                                                                                                                                                   | ✅ **Confirmé**                                    |
| **Consultation formations**   | 50% (Terminé)  | Pas de contenu formation visible dans ABBEAV                                                                                                                                                                                                                                         | ❌ **Non trouvé**                                  |
| **Suivi formations**          | 50% (Terminé)  | Pas de logique de suivi visible                                                                                                                                                                                                                                                      | ❌ **Non trouvé**                                  |

### 3.3 Projet TEATCHA — Fonctionnalités

| Fonctionnalité               | Statut rapport | Constat réel                                                                                                                                                                                                                                                                                                                                                                                                                           | Verdict                                                                                               |
| ---------------------------- | -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| **Création de compte**       | 10% (Terminé)  | [auth_service.dart](file:///home/atlas/Projects/WMK/teatcha/lib/services/auth_service.dart) (359 lignes) : Email/Password, Google Sign-In, Apple Sign-In **entièrement codés** + Firestore user storage                                                                                                                                                                                                                                | ✅ **Plus avancé que 10%**                                                                            |
| **Connexion au compte**      | 10% (Terminé)  | Login, forgot password, onboarding screens présents                                                                                                                                                                                                                                                                                                                                                                                    | ✅ **Plus avancé que 10%**                                                                            |
| **Consultation cours**       | 100% (Terminé) | [CourseController](file:///home/atlas/Projects/WMK/teatcha/lib/controllers/course_controller.dart#10-849) avec 8 cours **hardcodés en dummy** avec [\_generateDummyCourses()](file:///home/atlas/Projects/WMK/teatcha/lib/controllers/course_controller.dart#330-581). Aucun cours récupéré depuis Firebase                                                                                                                            | ⚠️ **100% données dummy — contrairement à ABBEAV, TEATCHA n'a pas de service backend pour les cours** |
| **Détails du cours**         | 100% (Terminé) | [course_detail_view.dart](file:///home/atlas/Projects/WMK/teatcha/lib/views/info/course_detail_view.dart) existe                                                                                                                                                                                                                                                                                                                       | ✅ **Confirmé**                                                                                       |
| **Consultation progression** | 50% (Terminé)  | [updateCourseProgress()](file:///home/atlas/Projects/WMK/teatcha/lib/controllers/course_controller.dart#264-308) codé dans le controller avec calcul de pourcentage. Logique locale seulement                                                                                                                                                                                                                                          | ⚠️ **Logique locale, pas persistée**                                                                  |
| **Gestion de profil**        | 80% (Avancé)   | [edit_profile_view.dart](file:///home/atlas/Projects/WMK/teatcha/lib/views/profile/edit_profile_view.dart), [settings_view.dart](file:///home/atlas/Projects/WMK/teatcha/lib/views/profile/settings_view.dart), [achievements_view.dart](file:///home/atlas/Projects/WMK/teatcha/lib/views/profile/achievements_view.dart), [certificates_view.dart](file:///home/atlas/Projects/WMK/teatcha/lib/views/profile/certificates_view.dart) | ✅ **Confirmé**                                                                                       |

---

## 4. POINTS CRITIQUES IDENTIFIÉS

### 4.1 ABBEAV : Architecture Solide mais Auth Désactivée

> [!NOTE]
> **Point positif majeur :** L'architecture d'ABBEAV est professionnelle avec 3 couches distinctes :
>
> - **MediaService** (1779 lignes) : Requêtes Supabase pures avec gestion d'erreurs PostgreSQL
> - **MediaRepository** (1193 lignes) : Stratégie cache-first avec Hive, revalidation en arrière-plan, et fallback réseau
> - **Providers Riverpod** : Gestion d'état réactive
>
> **Point d'attention :** L'auth guard dans le router est commenté (L.60-85 de router_app.dart), ce qui est courant en phase de développement mais doit être activé avant la mise en production.

### 4.2 TEATCHA : Données Cours 100% Dummy

> [!CAUTION]
> **Contrairement à ABBEAV**, le projet TEATCHA n'a pas de service backend pour les cours. Le [CourseController](file:///home/atlas/Projects/WMK/teatcha/lib/controllers/course_controller.dart) génère 8 cours avec `_generateDummyCourses()` en données statiques. **Aucun appel Firebase/Firestore** pour récupérer les cours. Cependant, l'auth (Email/Google/Apple) est bien connectée à Firebase.

### 4.3 L'Auth Guard est Commenté (ABBEAV)

Dans [router_app.dart](file:///home/atlas/Projects/WMK/abbeav/lib/core/router/router_app.dart#L60-L85), le bloc `redirect:` est **commenté**. Cela est courant en développement pour faciliter le debug, mais doit être réactivé.

### 4.4 Code Dupliqué (Commentaires)

Le fichier [main.dart](file:///home/atlas/Projects/WMK/abbeav/lib/main.dart) de `abbeav` contient une copie commentée (L.433-842). Le fichier router contient aussi une copie commentée (L.325-583). Le fichier [save/data.dart](file:///home/atlas/Projects/WMK/abbeav/lib/save/data.dart) est entièrement commenté (2225 lignes de code mort). Cela n'affecte pas le fonctionnement mais alourdit le codebase.

---

## 5. VALIDATION DES AFFIRMATIONS STRATÉGIQUES

| Affirmation                                      | Verdict                                                                                                                                        |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| "Blocages EXCLUSIVEMENT stratégiques/juridiques" | ⚠️ **Partiellement vrai pour ABBEAV** — L'architecture backend est connectée. Pour TEATCHA, des blocages techniques (données dummy) existent   |
| "Toutes les fonctionnalités des CDC développées" | ⚠️ **Partiellement vrai** — Les fonctionnalités média principales sont faites, mais réservation billets, votes, historique non trouvés         |
| "Application publiée sur App Store v1.2"         | ✅ **Crédible** — La version `1.0.2+7` est dans [pubspec.yaml](file:///home/atlas/Projects/WMK/abbeav/pubspec.yaml)                            |
| "Moteur de streaming fonctionnel"                | ✅ **Confirmé** — `video_player` + `chewie` intégrés, URLs depuis Supabase Storage                                                             |
| "Données proviennent de la base de données"      | ✅ **VRAI pour ABBEAV** — MediaService fait des requêtes Supabase PostgreSQL réelles. `save/data.dart` est entièrement commenté et non utilisé |
| "Stratégie Web + Synchronisation validée"        | ⚠️ **Le site web `abbeav-support` n'est qu'une landing page** avec 3 routes (home, privacy, terms)                                             |
| "Code NodeJS/TS existe en local"                 | ❌ **Non trouvé** — Aucun projet NodeJS/TS dans `/home/atlas/Projects/WMK/`                                                                    |
| "Problème paiements Apple IAP"                   | ✅ **Vrai** — L'argument sur les restrictions Apple pour les biens numériques est correct                                                      |
| "TEATCHA finalisée à 50%"                        | ⚠️ **L'auth est plus avancée que 10%**, mais les cours sont 100% dummy                                                                         |

---

## 6. QUALITÉ DU CODE

### Points Positifs

- ✅ Architecture 3 couches professionnelle dans `abbeav` (Service → Repository → Providers)
- ✅ Utilisation de Riverpod pour le state management (pattern moderne)
- ✅ Système de cache avec Hive sophistiqué (cache-first + stale-while-revalidate + revalidation en arrière-plan)
- ✅ Gestion d'erreurs structurée avec `ErrorHandler`, `AppLogger`, et ErrorBoundary (dartz `Either<Failure, T>`)
- ✅ Responsive design avec `ResponsiveHelper`
- ✅ Network monitoring avec bannière de connectivité
- ✅ Auth TEATCHA bien codée (Email, Google, Apple) avec Firestore
- ✅ MediaService de 1779 lignes avec requêtes Supabase complètes pour toutes les entités (media, cast, studios, seasons, episodes, similar_media)
- ✅ Modèles de données avec `fromJson()`/`toJson()` pour sérialisation

### Points Négatifs

- ❌ Code mort massif : `save/data.dart` (2225 lignes commentées), copies commentées dans main.dart et router_app.dart
- ❌ **Aucun test unitaire** (dossiers `test/` vides ou avec fichier par défaut)
- ❌ `.env` avec clés API commitées dans le code (risque de sécurité)
- ❌ Auth guard commenté = app non sécurisée en l'état
- ❌ TEATCHA : données cours 100% dummy, pas de service Firebase pour les cours

---

## 7. CONCLUSION & RECOMMANDATION

Le développeur TERI Armand a effectivement **travaillé sérieusement sur ces projets**, particulièrement sur ABBEAV qui présente une architecture professionnelle avec intégration Supabase réelle.

**Points forts confirmés :**

- L'architecture ABBEAV est **connectée à Supabase** (1779 lignes de service + 1193 lignes de repository avec cache)
- Le fichier `save/data.dart` est bien du **code mort commenté**, non utilisé par l'application
- Le player de streaming est fonctionnel
- L'auth TEATCHA est correctement intégrée à Firebase

**Points d'attention :**

- Plusieurs fonctionnalités (réservation, votes, historique) **n'ont pas été trouvées** dans le code
- TEATCHA : les cours sont encore en données dummy, pas connectés à Firebase
- Le site web est une simple landing page
- Le code NodeJS/TS mentionné n'existe pas dans le workspace fourni

> [!WARNING]
> **Estimation réelle d'avancement :**
>
> - **ABBEAV** : ~**55-65%** — Architecture solide et connectée à Supabase, streaming fonctionne, mais auth désactivée et certaines features non trouvées
> - **TEATCHA** : ~**30-35%** — L'auth est bien codée et connectée à Firebase, mais les cours sont 100% dummy, aucune intégration Firestore pour les contenus
