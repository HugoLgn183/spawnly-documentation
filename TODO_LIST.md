# OmniTech — Plan d'améliorations (TODO_LIST)

> Plan établi avec l'agent IA **Claude Code** (TP 2bis, partie C), à partir de la section « Limites connues et risques » de [`DOCUMENTATION.md`](DOCUMENTATION.md).
> Il s'adresse à un étudiant de **BTS SIO 1re année** : les actions sont simples, concrètes et réalisables en TP.

## Sommaire

1. [Cadre à respecter](#1-cadre-à-respecter)
2. [Comment lire ce plan](#2-comment-lire-ce-plan)
3. [Actions par thème](#3-actions-par-thème)
   - [3.1 Sécurité](#31-sécurité)
   - [3.2 Stockage des données](#32-stockage-des-données)
   - [3.3 Qualité du code](#33-qualité-du-code)
   - [3.4 Front-end](#34-front-end)
   - [3.5 Conformité / RGPD](#35-conformité--rgpd)
   - [3.6 Documentation](#36-documentation)
   - [3.7 Exploitation locale (WAMP)](#37-exploitation-locale-wamp)
4. [Définition de Terminé](#4-définition-de-terminé)
5. [Séquencement par jalons](#5-séquencement-par-jalons)

---

## 1. Cadre à respecter

Toutes les actions de ce plan respectent les règles suivantes :

- **WAMP local uniquement** : le site tourne sur Apache + PHP sous Windows, dans `C:\wamp64\www\OmniTech`. Aucun serveur distant, hébergeur, VPS, Docker, Nginx, PHP-FPM, nom de domaine réel ni déploiement automatisé.
- **Aucun paiement réel** : la commande reste une **simulation**. Aucune passerelle bancaire, aucune clé d'API bancaire, aucune saisie de carte.
- **Aucune nouvelle dépendance** : ni Composer, ni npm, ni framework. Une action qui en demanderait une est marquée « **sous accord du professeur** ». *(Aucune action de ce plan n'en demande.)*
- **Stockage JSON** : le stockage actuel dans des fichiers JSON est **adapté au projet** et n'est pas un défaut. Une migration vers **SQLite** est proposée uniquement en option (P2).
- **Le site doit rester fonctionnel** après chaque tâche.

---

## 2. Comment lire ce plan

| Colonne | Signification |
|---|---|
| **Priorité** | **P0** = critique, à faire en premier · **P1** = important · **P2** = optionnel, pour aller plus loin |
| **Complexité** | **Simple** (moins d'1 h, 1 fichier) · **Moyen** (1 à 2 h, quelques fichiers) · **Complexe** (une séance ou plus) |
| **Critère d'acceptation** | Ce que vous devez pouvoir **constater vous-même** dans le navigateur ou dans un fichier pour valider la tâche |
| **Fichiers concernés** | Les fichiers du site à modifier |

Le numéro entre crochets, par exemple [Doc §10.1], renvoie au point correspondant dans `DOCUMENTATION.md`.

---

## 3. Actions par thème

### 3.1 Sécurité

| # | Action | Priorité | Complexité | Fichiers concernés | Critère d'acceptation |
|---|---|---|---|---|---|
| S1 | **Interdire l'accès aux fichiers `.md` (`CLAUDE.md`, dossier `Docs TP2 bis/`) et au dossier `lib/` par le navigateur** [Doc §10.1, §10.8] | P0 | Simple | `.htaccess` (racine, règle `FilesMatch "\.md$"` valable aussi dans les sous-dossiers), nouveau `lib/.htaccess` | `http://localhost/OmniTech/CLAUDE.md`, `…/Docs%20TP2%20bis/DOCUMENTATION.md` et `…/lib/phpmailer/PHPMailer.php` affichent **« Forbidden » (erreur 403)**. Le site et l'envoi des mails fonctionnent toujours. |
| S2 | **Temps de réponse identique dans « Mot de passe oublié »** [Doc §10.3] | P0 | Moyen | `client/forgot-password.php` | En demandant un code pour un compte **existant** puis pour une adresse **inconnue**, la réponse arrive après un délai comparable (à mesurer avec l'onglet *Réseau* des outils de développement, touche F12). |
| S3 | **Ajouter un champ piège et un délai à l'inscription** [Doc §10.5] | P1 | Simple | `client/register.php`, `js/client-auth.js` | Deux inscriptions depuis la même IP à moins de 30 s : la seconde est refusée avec un message clair. Un formulaire envoyé avec le champ piège rempli ne crée aucun compte. |
| S4 | **Limiter le formulaire de contact par adresse IP** (et plus seulement par session) [Doc §10.6] | P1 | Moyen | `contact.php`, `admin/inc/auth.php` | Après suppression du cookie de session, un second message envoyé en moins de 30 s est **toujours refusé**. |
| S5 | **Ajouter des en-têtes de sécurité HTTP** [Doc §10.7] | P1 | Simple | `.htaccess` (racine) | Dans F12 → *Réseau* → une page → *En-têtes de réponse* : présence de `X-Frame-Options: DENY` et `X-Content-Type-Options: nosniff`. Toutes les pages s'affichent normalement. |
| S6 | **Message neutre à l'inscription** quand l'adresse est déjà prise [Doc §10.4] | P2 | Moyen | `client/register.php` | Le message ne permet plus de savoir si l'adresse possède déjà un compte (par exemple : « Si cette adresse est disponible, votre compte a été créé ; sinon, utilisez Mot de passe oublié »). |
| S7 | **Politique de sécurité du contenu (CSP)** en mode « rapport uniquement » | P2 | Complexe | `.htaccess`, pages HTML | L'en-tête `Content-Security-Policy-Report-Only` est présent et **aucune erreur CSP** n'apparaît dans la console (F12) sur les pages du site et du panel. |

> **Termes techniques**
> - *En-tête HTTP* : information envoyée par le serveur avec chaque page, invisible à l'écran. `X-Frame-Options` empêche d'afficher le site dans un cadre d'un autre site (attaque dite de *clickjacking*).
> - *Champ piège* (*honeypot*) : champ caché que seul un robot remplit ; s'il est rempli, on ignore l'envoi.
> - *CSP* : liste des sources autorisées (scripts, images…) ; en mode *Report-Only*, le navigateur signale les problèmes sans rien bloquer.

### 3.2 Stockage des données

| # | Action | Priorité | Complexité | Fichiers concernés | Critère d'acceptation |
|---|---|---|---|---|---|
| D1 | **Ne jamais sauvegarder ni publier le mot de passe SMTP** [Doc §10.2, §10.10] | P0 | Simple | nouveau `.gitignore`, `data/` | Un fichier `.gitignore` exclut `data/*.json` (sauf modèles) et `storage/`. Une copie `data/email_config.example.json` existe, **sans mot de passe**. Le site fonctionne toujours. |
| D2 | **Verrouiller les fichiers JSON pendant la lecture et l'écriture** [Doc §10.9] | P1 | Moyen | `admin/inc/auth.php` (`ot_write_json`, nouvelle fonction de mise à jour) | Les journaux, compteurs anti-brute-force et messages de contact passent par une fonction qui pose un verrou (`flock`) pendant tout le cycle *lire → modifier → écrire*. Les journaux s'enregistrent toujours correctement. |
| D3 | **Corriger la rubrique « API » de la sauvegarde** [Doc §10.19] | P1 | Simple | `admin/maintenance.php` | Le fichier exporté par Maintenance → Sauvegarde indique le **moteur réellement actif** (`smtp`) et ne contient **aucun mot de passe**. |
| D4 | **(Option) Migrer les comptes clients vers SQLite** | P2 | Complexe | `inc/client_auth.php`, nouveau `data/omnitech.sqlite` | Inscription, connexion et mot de passe oublié fonctionnent avec la base SQLite. Les comptes existants sont repris. **À valider avec le professeur avant de commencer.** |

> **Termes techniques**
> - *`.gitignore`* : liste des fichiers que Git ne doit pas envoyer sur GitHub.
> - *Verrou* (`flock`) : empêche deux requêtes de modifier le même fichier en même temps.
> - *SQLite* : base de données contenue dans un seul fichier, déjà intégrée à PHP sous WAMP (extension `pdo_sqlite`) ; aucune installation n'est nécessaire.

### 3.3 Qualité du code

| # | Action | Priorité | Complexité | Fichiers concernés | Critère d'acceptation |
|---|---|---|---|---|---|
| Q1 | **Supprimer le code mort** [Doc §10.11] | P1 | Simple | `admin/inc/auth.php`, `inc/client_auth.php` | `ot_user_add`, `ot_user_delete` et `ot_client_find_by_id` n'existent plus. `php -l` ne signale aucune erreur et toutes les pages du panel s'ouvrent. |
| Q2 | **Afficher les produits de l'accueil depuis `products.json`** [Doc §10.12] | P1 | Moyen | `index.html`, `js/script.js`, `products.json` | Modifier le prix d'un produit dans `products.json` change le prix **à la fois** sur l'accueil et dans le catalogue. |
| Q3 | **Script de vérification syntaxique** de tous les fichiers PHP | P1 | Simple | nouveau `verifier.bat` | Un double-clic sur `verifier.bat` lance `php -l` sur chaque fichier PHP du site et affiche « No syntax errors » pour chacun. |
| Q4 | **Numéro de version CSS/JS centralisé** [Doc §10.14] | P2 | Moyen | pages HTML/PHP, `admin/inc/layout.php` | Le numéro de version n'est plus écrit qu'à **un seul endroit** ; le changer met à jour toutes les pages. |
| Q5 | **Mutualiser l'en-tête et le pied de page** [Doc §10.13] | P2 | Complexe | `index.html`, `produits.html`, `contact.php` (passage en `.php` + `include`) | Ajouter un lien dans le menu ne demande de modifier **qu'un seul fichier**. Les anciennes adresses `.html` redirigent vers les nouvelles. |

### 3.4 Front-end

| # | Action | Priorité | Complexité | Fichiers concernés | Critère d'acceptation |
|---|---|---|---|---|---|
| F1 | **Rendre la newsletter honnête** [Doc §10.18] | P1 | Simple | `index.html`, `produits.html`, `contact.php`, `js/script.js` | Soit les adresses sont réellement enregistrées (fichier JSON + affichage dans le panel), soit le message indique clairement « fonctionnalité de démonstration ». |
| F2 | **Tableaux du panel lisibles sur mobile** | P1 | Moyen | `admin/assets/admin.css` | En mode mobile (F12 → icône téléphone, largeur 375 px), les journaux s'affichent sans défilement horizontal de la page. |
| F5 | **En-tête du site sur mobile** : le bouton « Connexion », le panier et le menu se serrent au bord de l'écran (constaté pendant R2) | P1 | Simple | `css/style.css` | En largeur 375 px (F12 → icône téléphone), les trois éléments de l'en-tête sont entièrement visibles et cliquables, sur toutes les pages. |
| F3 | **Bascule thème clair / sombre** pour le site | P2 | Moyen | `css/style.css`, `css/brand.css`, `js/script.js` | Un bouton dans l'en-tête change le thème ; le choix est conservé après rechargement (`localStorage`). |
| F4 | **Vérification d'accessibilité** avec l'outil *Lighthouse* intégré à Chrome | P2 | Simple | selon les résultats | Le score « Accessibilité » de Lighthouse est d'au moins **90** sur l'accueil, le catalogue et le contact. |

### 3.5 Conformité / RGPD

| # | Action | Priorité | Complexité | Fichiers concernés | Critère d'acceptation |
|---|---|---|---|---|---|
| R1 | **Ajouter une mention d'information** sous le formulaire de contact et à l'inscription [Doc §10.16] | P0 | Simple | `contact.php`, `js/client-auth.js` | Une phrase explique quelles données sont conservées (nom, e-mail, message, adresse IP), pourquoi, et combien de temps. |
| R2 | **Créer une page « Mentions légales et confidentialité »** | P1 | Simple | nouvelle page `confidentialite.html`, pieds de page | La page est accessible depuis le pied de page de toutes les pages et précise que le site est **fictif et pédagogique**. |
| R3 | **Durée de conservation des messages de contact** | P1 | Moyen | `admin/inc/auth.php` (`ot_contact_log`), `admin/maintenance.php` | Les messages de plus de 6 mois (durée à fixer) sont supprimés automatiquement ; la durée est affichée dans Maintenance → Journaux. |
| R4 | **Page « Mon compte » client** : modifier son nom, supprimer son compte [Doc §10.17] | P2 | Complexe | nouveau `client/account.php`, `js/client-auth.js`, `inc/client_auth.php` | Un client connecté peut supprimer son compte après confirmation par mot de passe ; il ne peut plus se connecter ensuite. |

> **Terme technique** — *RGPD* (Règlement général sur la protection des données) : règlement européen qui impose d'informer les personnes sur l'usage de leurs données, de limiter leur durée de conservation et de permettre leur suppression.

### 3.6 Documentation

| # | Action | Priorité | Complexité | Fichiers concernés | Critère d'acceptation |
|---|---|---|---|---|---|
| M1 | **Créer `PROMPTS.md`** avec les consignes exactes données à l'agent IA | P0 | Simple | nouveau `PROMPTS.md` | Le fichier contient au moins les consignes des parties A, C et E du TP 2bis, et il est présent dans le dépôt GitHub. |
| M2 | **Créer un `README.md` d'installation** | P1 | Simple | nouveau `README.md` | Un camarade peut installer le site sur son WAMP en suivant uniquement le README (copie du dossier, hôte virtuel, configuration des e-mails). |
| M3 | **Tenir `DOCUMENTATION.md` à jour** après chaque tâche | P1 | Simple | `DOCUMENTATION.md` | La section « Limites connues » ne cite plus les points corrigés. |

### 3.7 Exploitation locale (WAMP)

| # | Action | Priorité | Complexité | Fichiers concernés | Critère d'acceptation |
|---|---|---|---|---|---|
| E1 | **Activer l'hôte virtuel `omnitech.local`** (déjà configuré dans Apache) | P0 | Simple | fichier `hosts` de Windows (droits administrateur), redémarrage d'Apache | `http://omnitech.local/` affiche l'accueil, et la connexion admin fonctionne à cette adresse. |
| E2 | **Vider le journal de connexion avant le rendu** (comptes de test) | P0 | Simple | Panel → Maintenance → Journaux | Le journal ne contient plus de comptes `client.test.*` ni `personne.test`. |
| E3 | **Procédure de sauvegarde locale** du dossier `data/` | P1 | Simple | nouveau `sauvegarde.bat` | Un double-clic copie `data/` dans un dossier daté (par exemple `C:\sauvegardes\omnitech-2026-10-15`) ; la restauration est décrite dans le README. |
| E4 | **Activer les logs de débogage des mails** et vérifier qu'ils ne contiennent aucun secret | P1 | Simple | Panel → Configuration des e-mails | Après un « Tester l'envoi », `storage/logs/mail.log` contient la trace SMTP avec `[credentials hidden]` et `[contenu du message masqué]`. |

---

## 4. Définition de Terminé

Une tâche est **terminée** seulement si **tous** les points suivants sont vrais :

1. Le **critère d'acceptation** de la tâche est vérifié dans le navigateur (WAMP local) ou dans le fichier concerné.
2. `php -l` ne signale **aucune erreur de syntaxe** dans les fichiers PHP modifiés.
3. Le **parcours de non-régression** fonctionne toujours :
   - l'accueil, le catalogue (recherche, filtres, fiche produit) et le panier s'affichent sans erreur dans la console (F12) ;
   - le formulaire de contact envoie un message ;
   - la connexion **client** reste sur le site, la connexion **admin** ouvre le panel ;
   - toutes les pages du panel s'ouvrent.
4. **Aucune contrainte n'est enfreinte** : WAMP local, paiement simulé, aucune nouvelle dépendance.
5. **Aucun secret** (mot de passe, code de réinitialisation) n'apparaît à l'écran, dans un log ou dans un fichier publié.
6. La tâche est **cochée** dans ce fichier et `DOCUMENTATION.md` est mis à jour si besoin.

---

## 5. Séquencement par jalons

Chaque jalon se termine par le parcours de non-régression de la « Définition de Terminé ».

### Jalon 1 — Corrections critiques (P0) · environ 1 séance
- [ ] S1 — Interdire les fichiers `.md` et `lib/` par HTTP *(fait le 15/10/2026 — 5 adresses vérifiées en 403, site et PHPMailer fonctionnels)*
- [ ] D1 — `.gitignore` + modèle de configuration sans mot de passe *(fait le 15/10/2026 — testé dans un dépôt Git jetable : seuls `data/.htaccess` et `data/email_config.example.json` seraient publiés)*
- [ ] R1 — Mentions d'information sur les formulaires *(fait le 15/10/2026 — sous le formulaire de contact et sous « Créer un compte »)*
- [ ] S2 — Temps de réponse identique dans « Mot de passe oublié » *(fait le 15/10/2026 — réponse envoyée avant le mail : ~0,36 s dans tous les cas)*
- [ ] M1 — `PROMPTS.md` *(fait le 15/10/2026 — consignes des parties A, C et E + annexe TP 2, dans `Docs TP2 bis/`)*
- [ ] E1 — Activer `omnitech.local`
- [ ] E2 — Vider le journal de connexion *(à faire en dernier, juste avant le rendu)*

### Jalon 2 — Robustesse et conformité (P1, sécurité et données) · environ 1 à 2 séances
- [ ] S3 — Protection de l'inscription *(fait le 15/10/2026 — champ piège + 30 s par IP, stocké côté serveur ; refus enregistrés dans le journal, filtre « Échecs / blocages »)*
- [ ] S4 — Limite du contact par IP *(fait le 15/10/2026 — 30 s par IP côté serveur, fonction commune `ot_rate_wait` / `ot_rate_mark` ; refus et robots dans « Échecs / blocages »)*
- [ ] S5 — En-têtes de sécurité *(fait le 15/10/2026 — X-Frame-Options, X-Content-Type-Options, Referrer-Policy, Permissions-Policy ; vérifiés sur 11 types de réponses, console sans erreur)*
- [ ] D2 — Verrou sur les fichiers JSON *(fait le 15/10/2026 — `ot_json_update` ; test 10 processus simultanés : 500/500 évènements contre 22/500 avant ; noms temporaires uniques)*
- [ ] D3 — Rubrique « API » de la sauvegarde *(fait le 15/10/2026 — moteur réellement actif exporté, aucun secret dans l'export)*
- [ ] R2 — Page « Mentions légales et confidentialité » *(fait le 15/10/2026 — `confidentialite.html`, lien dans le pied de page des 4 pages)*
- [ ] R3 — Durée de conservation des messages *(fait le 15/10/2026 — 6 mois, purge automatique journalisée, durée affichée dans Maintenance → Journaux)*

### Jalon 3 — Qualité, documentation, exploitation (P1) · environ 1 séance
- [ ] Q1 — Supprimer le code mort *(fait le 15/10/2026 — `ot_user_add`, `ot_user_delete`, `ot_client_find_by_id` supprimées ; 9 pages du panel et 5 du site vérifiées)*
- [ ] Q2 — Produits de l'accueil depuis `products.json` *(fait le 15/10/2026 — `data-ids="1,3,4"` + `buildProductCard` commune ; test : prix modifié une seule fois, visible sur les deux pages)*
- [ ] Q3 — `verifier.bat` *(fait le 15/10/2026 — 19 fichiers vérifiés avec la version de PHP d'Apache ; erreur volontaire bien détectée ; bloqué par HTTP)*
- [ ] F1 — Newsletter honnête *(fait le 15/10/2026 — option « démonstration » choisie : mention avant l'envoi + message honnête après, sur les 3 pages)*
- [ ] F2 — Tableaux du panel sur mobile *(fait le 15/10/2026 — sous 640 px, chaque ligne devient une carte avec libellés ; à 375 px : 0 défilement horizontal sur les 7 tableaux du panel, affichage ordinateur inchangé)*
- [ ] F5 — En-tête du site sur mobile *(fait le 15/10/2026 — logo à gauche, Connexion / panier / menu à droite en 42 px ; logo réduit à son icône sous 420 px (choix : garder « Connexion » en toutes lettres) ; menu du compte calé dans l'écran ; vérifié à 320, 375, 421 et 720 px sur les 4 pages ; sur ordinateur, liens centrés et panier groupé avec Connexion, vérifié de 721 à 1280 px)*
- [ ] M2 — `README.md`
- [ ] M3 — Mise à jour de `DOCUMENTATION.md`
- [ ] E3 — `sauvegarde.bat`
- [ ] E4 — Logs de débogage des mails

### Jalon 4 — Pour aller plus loin (P2, optionnel)
- [ ] S6 — Message neutre à l'inscription
- [ ] S7 — CSP en mode rapport
- [ ] Q4 — Version CSS/JS centralisée
- [ ] Q5 — En-tête et pied de page mutualisés
- [ ] F3 — Thème clair / sombre
- [ ] F4 — Score Lighthouse ≥ 90
- [ ] R4 — Page « Mon compte »
- [ ] D4 — Migration SQLite *(sous accord du professeur)*
