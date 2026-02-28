# RAPPORT DE CONTRE-AUDIT TECHNIQUE — Projets ABBEAV & TEATCHA

**Date :** 28 Février 2026  
**Auditeur :** Audit Indépendant (analyse du code source)  
**Objectif :** Valider ou infirmer le rapport technique de TERI Armand (25/02/2026)

---

## 1. RÉSUMÉ EXÉCUTIF

L'analyse indépendante du code source confirme **partiellement** le rapport du développeur. Plusieurs affirmations sont vérifiées, mais **des nuances importantes** et **certaines exagérations** méritent d'être signalées.

> [!IMPORTANT]
> **Verdict global :** Le développeur a fait un travail réel et structuré, mais les pourcentages d'avancement annoncés dans son rapport sont **gonflés** pour certaines fonctionnalités. De plus, une partie significative du "code fonctionnel" repose sur des **données dummy/statiques** et non sur une intégration backend réelle.

---

## 2. INVENTAIRE DES PROJETS TROUVÉS

| Projet | Technologie | Fichiers Source | Commits Git | Description |
|--------|------------|----------------|-------------|-------------|
| `abbeav` | Flutter + Supabase + Riverpod | **80 fichiers Dart** | N/A (accès refusé) | Version principale sur App Store |
| `abbeav-v1` | Flutter + Supabase + Provider | **40 fichiers Dart** | **1 seul commit** | Version d'extension/codes à activer |
| `teatcha` | Flutter + Firebase + Provider | **47 fichiers Dart** | N/A (accès refusé) | App éducative |
| `abbeav-support` | React + Vite + TailwindCSS | **66 fichiers TSX** | **1 seul commit** | Page web de support (landing) |

---

## 3. VALIDATION POINT PAR POINT

### 3.1 Architecture Technique

| Affirmation du rapport | ✅/❌ | Constat réel |
|------------------------|-------|--------------|
| Flutter (Dart) cross-platform | ✅ **VRAI** | Confirmé - SDK ^3.8.1, configs iOS + Android présentes |
| Supabase comme BaaS (ABBEAV) | ✅ **VRAI** | Confirmé - `supabase_flutter: ^2.12.0`, .env avec URL et clé anon |
| PostgreSQL via Supabase | ✅ **VRAI** | Fichiers SQL ([commands.sql](file:///home/atlas/Projects/WMK/abbeav/commands.sql), [insert.sql](file:///home/atlas/Projects/WMK/abbeav/insert.sql)) présents dans `abbeav` |
| Supabase Auth (passwordless) | ⚠️ **PARTIELLEMENT** | Auth codée mais **auth guard commenté** dans le router. Le système d'auth n'est pas actif |
| Supabase Storage + CDN | ✅ **VRAI** | URLs Supabase S3 signées confirmées dans tout le code (`supabase.co/storage/v1/object/sign/`) |
| Firebase pour TEATCHA | ✅ **VRAI** | `firebase_core`, `cloud_firestore`, `firebase_auth` dans les dépendances |
| Architecture Serverless Full BaaS | ⚠️ **PARTIELLEMENT** | L'architecture est en place mais **fortement dépendante de données statiques** |

### 3.2 Projet ABBEAV — Fonctionnalités

| Fonctionnalité | Statut rapport | Constat réel | Verdict |
|---------------|---------------|-------------|---------|
| **Inscription/Connexion** | 50% (Terminé) | Auth service codé, OTP (pinput), mais **auth guard commenté** dans [router_app.dart](file:///home/atlas/Projects/WMK/abbeav/lib/core/router/router_app.dart) L.60-85. L'app tourne sans authentification | ⚠️ **Code présent mais désactivé** |
| **Consultation films/séries** | 100% (Terminé) | Écran `MediaHomeScreen` fonctionnel. **MAIS** : les données viennent d'un fichier statique [save/data.dart](file:///home/atlas/Projects/WMK/abbeav/lib/save/data.dart) (2225 lignes de données hardcodées), pas d'une vraie API/BDD Supabase | ⚠️ **Fonctionne avec données dummy** |
| **Détails films** | 100% (Terminé) | `MediaDetailScreen` existe et reçoit `mediaId` | ✅ **Confirmé** |
| **Lecture Streaming** | 100% (Terminé) | `video_player` + `chewie` intégrés. `MediaPlayerScreen` existe avec URLs vidéo Supabase S3. Vidéos de **6 secondes** chacune | ⚠️ **Player fonctionne, mais vidéos = clips de 6s** |
| **Fiches Acteurs** | 100% (Terminé) | 10 acteurs fictifs hardcodés dans [data.dart](file:///home/atlas/Projects/WMK/abbeav/lib/save/data.dart) avec avatars, biographies, galeries | ⚠️ **Données 100% statiques/fictives** |
| **Fiches Studios** | 100% (Terminé) | `AboutBrandScreen` existe, route configurée | ✅ **Confirmé** |
| **Historique visionnage** | 50% (Terminé) | Pas de code d'historique visible dans les routes actives | ❌ **Non vérifié** |
| **Réservation billets** | 50% (Terminé) | **Aucun code** de réservation visible dans le codebase | ❌ **Non trouvé dans le code** |
| **Votes œuvres** | 50% (Terminé) | Pas de système de vote visible dans les fichiers | ❌ **Non trouvé dans le code** |
| **Notes et commentaires** | 50% (Terminé) | `abbeav-v1` a [reviews_all_service.dart](file:///home/atlas/Projects/WMK/abbeav-v1/lib/services/review/reviews_all_service.dart) et [reviews_all_screen.dart](file:///home/atlas/Projects/WMK/abbeav-v1/lib/view/review/reviews_all_screen.dart) mais pas dans la version principale | ⚠️ **Existe dans -v1 seulement** |
| **Gestion de profils** | 80% (Avancé) | `ProfileScreen`, `ProfileEditScreen`, `ProfilePasswordScreen` existent | ✅ **Confirmé** |
| **Playlists et favoris** | 50% (Terminé) | `WatchlistScreen` + route existent | ✅ **Confirmé** |
| **Consultation formations** | 50% (Terminé) | Pas de contenu formation visible dans ABBEAV | ❌ **Non trouvé** |
| **Suivi formations** | 50% (Terminé) | Pas de logique de suivi visible | ❌ **Non trouvé** |

### 3.3 Projet TEATCHA — Fonctionnalités

| Fonctionnalité | Statut rapport | Constat réel | Verdict |
|---------------|---------------|-------------|---------|
| **Création de compte** | 10% (Terminé) | [auth_service.dart](file:///home/atlas/Projects/WMK/teatcha/lib/services/auth_service.dart) (359 lignes) : Email/Password, Google Sign-In, Apple Sign-In **entièrement codés** + Firestore user storage | ✅ **Plus avancé que 10%** |
| **Connexion au compte** | 10% (Terminé) | Login, forgot password, onboarding screens présents | ✅ **Plus avancé que 10%** |
| **Consultation cours** | 100% (Terminé) | [CourseController](file:///home/atlas/Projects/WMK/teatcha/lib/controllers/course_controller.dart#10-849) avec 8 cours **hardcodés en dummy** avec [_generateDummyCourses()](file:///home/atlas/Projects/WMK/teatcha/lib/controllers/course_controller.dart#330-581). Aucun cours récupéré depuis Firebase | ⚠️ **100% données dummy** |
| **Détails du cours** | 100% (Terminé) | [course_detail_view.dart](file:///home/atlas/Projects/WMK/teatcha/lib/views/info/course_detail_view.dart) existe | ✅ **Confirmé** |
| **Consultation progression** | 50% (Terminé) | [updateCourseProgress()](file:///home/atlas/Projects/WMK/teatcha/lib/controllers/course_controller.dart#264-308) codé dans le controller avec calcul de pourcentage. Logique locale seulement | ⚠️ **Logique locale, pas persistée** |
| **Gestion de profil** | 80% (Avancé) | [edit_profile_view.dart](file:///home/atlas/Projects/WMK/teatcha/lib/views/profile/edit_profile_view.dart), [settings_view.dart](file:///home/atlas/Projects/WMK/teatcha/lib/views/profile/settings_view.dart), [achievements_view.dart](file:///home/atlas/Projects/WMK/teatcha/lib/views/profile/achievements_view.dart), [certificates_view.dart](file:///home/atlas/Projects/WMK/teatcha/lib/views/profile/certificates_view.dart) | ✅ **Confirmé** |

---

## 4. POINTS CRITIQUES IDENTIFIÉS

### 4.1 Données Hardcodées vs Backend Réel

> [!CAUTION]
> **Le problème le plus sérieux :** Le rapport donne l'impression que les fonctionnalités sont connectées à un backend fonctionnel. En réalité :
> - **ABBEAV** : Le fichier [save/data.dart](file:///home/atlas/Projects/WMK/abbeav/lib/save/data.dart) (2225 lignes, 203KB) contient TOUTES les données (acteurs, films, séries, studios, épisodes) en dur dans le code. Les URLs Supabase S3 sont des **tokens signés avec expiration** (exp: 2026-2027).
> - **TEATCHA** : Le [CourseController](file:///home/atlas/Projects/WMK/teatcha/lib/controllers/course_controller.dart#10-849) génère des cours avec [_generateDummyCourses()](file:///home/atlas/Projects/WMK/teatcha/lib/controllers/course_controller.dart#330-581) en données statiques. **Aucun appel Firebase/Firestore** pour récupérer les cours.

### 4.2 L'Auth Guard est Commenté (ABBEAV)

Dans [router_app.dart](file:///home/atlas/Projects/WMK/abbeav/lib/core/router/router_app.dart#L60-L85), tout le bloc `redirect:` qui gère l'authentification est **commenté** (L.60-85). Cela signifie que l'app tourne sans vérification d'authentification.

### 4.3 Vidéos de Démonstration

Les vidéos hébergées sur Supabase S3 sont des clips de **6 secondes** (`duration: '0m06s'`). Ce sont des fichiers de démonstration, pas du contenu réel de streaming.

### 4.4 Tokens Signés avec Expiration

Tous les URLs des médias utilisent des tokens JWT signés avec des dates d'expiration (~1 an). Ces URLs **expireront** et nécessiteront une régénération.

### 4.5 Code Dupliqué

Le fichier [main.dart](file:///home/atlas/Projects/WMK/abbeav/lib/main.dart) de `abbeav` contient **le même code deux fois** (L.1-431 puis L.433-842 en commentaire). Le fichier router contient aussi une copie commentée (L.325-583).

---

## 5. VALIDATION DES AFFIRMATIONS STRATÉGIQUES

| Affirmation | Verdict |
|------------|---------|
| "Blocages EXCLUSIVEMENT stratégiques/juridiques" | ⚠️ **Partiellement vrai** — Les blocages techniques existent aussi (données dummy, auth désactivée) |
| "Toutes les fonctionnalités des CDC développées" | ❌ **Exagéré** — Réservation billets, votes, historique non trouvés dans le code |
| "Application publiée sur App Store v1.2" | ✅ **Crédible** — La version `1.0.2+7` est dans [pubspec.yaml](file:///home/atlas/Projects/WMK/abbeav/pubspec.yaml) |
| "Moteur de streaming fonctionnel" | ⚠️ **Le player fonctionne**, mais les vidéos sont des clips de 6s |
| "Stratégie Web + Synchronisation validée" | ⚠️ **Le site web `abbeav-support` n'est qu'une landing page** avec 3 routes (home, privacy, terms) — pas de système de paiement, pas de CMS |
| "Code NodeJS/TS existe en local" | ❌ **Non trouvé** — Aucun projet NodeJS/TS dans `/home/atlas/Projects/WMK/` |
| "Problème paiements Apple IAP" | ✅ **Vrai** — L'argument sur les restrictions Apple pour les biens numériques est correct |
| "TEATCHA finalisée à 50%" | ⚠️ **L'auth est plus avancée que 10%**, mais les cours sont 100% dummy |

---

## 6. QUALITÉ DU CODE

### Points Positifs
- ✅ Architecture modulaire propre dans `abbeav` (core/modules/auth/media/profile)
- ✅ Utilisation de Riverpod pour le state management (pattern moderne)
- ✅ Système de cache avec Hive bien intégré
- ✅ Gestion d'erreurs structurée (`ErrorHandler`, `AppLogger`)
- ✅ Responsive design avec `ResponsiveHelper`
- ✅ Network monitoring avec bannière de connectivité
- ✅ Auth Teatcha bien codée (Email, Google, Apple)

### Points Négatifs
- ❌ Code dupliqué massivement (main.dart, router_app.dart contiennent des copies commentées)
- ❌ **Aucun test unitaire** (dossiers `test/` vides ou avec fichier par défaut)
- ❌ Données hardcodées au lieu d'intégration backend réelle
- ❌ `.env` avec clés API commitées dans le code (risque de sécurité)
- ❌ Auth guard commenté = app non sécurisée
- ❌ Tokens S3 avec expiration = maintenance nécessaire

---

## 7. CONCLUSION & RECOMMANDATION

Le développeur TERI Armand a effectivement **travaillé sur ces projets** et produit du code structuré. L'architecture est correcte et les choix techniques (Flutter, Supabase, Firebase) sont pertinents.

**Cependant**, le rapport **surestime l'avancement** :
- Les pourcentages de "fonctionnalités terminées" sont gonflés car beaucoup reposent sur des **données statiques/dummy**
- Plusieurs fonctionnalités mentionnées comme "terminées mais désactivées" (réservation, votes, historique) **n'ont pas été trouvées** dans le code
- La version web mentionnée comme "existante en NodeJS/TS local" **n'existe pas** dans le workspace fourni
- Le "code à activer" dans `abbeav-v1` n'a qu'**un seul commit** et est une version simplifiée

> [!WARNING]
> **Estimation réelle d'avancement :**
> - **ABBEAV** : ~**40-50%** (pas 70% comme annoncé) — L'UI et le player marchent, mais tout repose sur des données statiques, l'auth est désactivée, et plusieurs features sont manquantes
> - **TEATCHA** : ~**30-35%** (pas 50%) — L'auth est bien codée, mais les cours sont 100% dummy, aucune intégration Firestore pour les contenus
