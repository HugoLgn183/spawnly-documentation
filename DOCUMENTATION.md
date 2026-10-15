# OmniTech — Documentation technique

> Analyse du site réalisée avec l'agent IA **Claude Code** (TP 2bis, partie A), à partir du code réellement présent dans `C:\wamp64\www\OmniTech`.
> Chaque affirmation renvoie au fichier qui la justifie. Quand une information n'est pas déterminable depuis le code, c'est indiqué.
> Date de l'analyse : 15/10/2026.

## Table des matières

1. [Présentation générale du site](#1-présentation-générale-du-site)
2. [Stack technique](#2-stack-technique)
3. [Arborescence du projet](#3-arborescence-du-projet)
4. [Points d'entrée et routage](#4-points-dentrée-et-routage)
5. [Stockage des données](#5-stockage-des-données)
6. [Authentification, sessions et sécurité](#6-authentification-sessions-et-sécurité)
7. [Front-end](#7-front-end)
8. [Intégrations externes](#8-intégrations-externes)
9. [Configuration](#9-configuration)
10. [Limites connues et risques](#10-limites-connues-et-risques)
11. [Points forts](#11-points-forts)

---

## 1. Présentation générale du site

OmniTech est le site vitrine d'une **entreprise fictive** de vente de périphériques informatiques (claviers, souris, écrans, audio, webcams, stockage, réseau). Il a été réalisé dans le cadre des TP de BTS SIO et fonctionne **uniquement en local sous WAMP**.

Fonctionnalités principales :

| Fonctionnalité | Description | Fichiers |
|---|---|---|
| Accueil | Bannière, arguments, 3 produits mis en avant (chargés depuis `products.json`), avis clients, chiffres clés, newsletter | `index.html`, `js/script.js` (`initFeatured`) |
| Catalogue | 12 produits chargés en AJAX, recherche en temps réel, filtres par catégorie | `produits.html`, `products.json`, `js/script.js` |
| Fiche produit | Fenêtre de détail : grande photo, prix, quantité, fiche technique | `js/script.js` (`openProductDetail`) |
| Panier | Tiroir latéral, quantités, livraison offerte dès 80 €, **commande simulée** (aucun paiement) | `js/script.js` (localStorage) |
| Contact | Formulaire envoyant un vrai e-mail au destinataire configuré, copie archivée | `contact.php` |
| Espace client | Inscription, connexion, déconnexion, mot de passe oublié | `client/*.php`, `js/client-auth.js` |
| Connexion unique | Même fenêtre pour clients et administrateurs ; l'admin est redirigé vers le panel | `client/login.php` |
| Panel d'administration | Tableau de bord, configuration des e-mails, mot de passe, adresse e-mail, maintenance | `admin/*.php` |

---

## 2. Stack technique

| Élément | Valeur | Source |
|---|---|---|
| Serveur web | Apache 2.4.65 (WAMP, Windows) | journaux Apache de WAMP |
| Langage serveur | PHP 8.3.28 | journaux Apache (`PHP/8.3.28`) |
| Langages front | HTML5, CSS3, JavaScript (ES5, sans framework) | `*.html`, `css/`, `js/` |
| Stockage | Fichiers **JSON** (aucune base de données) | `data/*.json`, `admin/inc/auth.php` |
| Bibliothèque PHP | PHPMailer 7.1.1 (3 fichiers copiés, sans Composer) | `lib/phpmailer/SMTP.php` (`VERSION = '7.1.1'`) |
| Bibliothèques JS / CSS | Aucune (ni jQuery, ni Bootstrap, ni CDN) | `index.html`, `produits.html`, `contact.php` |
| Base MySQL | Fournie par WAMP mais **non utilisée** par le site | aucun appel `mysqli` / `PDO` dans le code |

---

## 3. Arborescence du projet

```
OmniTech/
├── index.html              Accueil
├── produits.html           Catalogue (rempli par JavaScript)
├── contact.php             Formulaire de contact (traitement PHP + envoi mail)
├── contact.html            Redirection vers contact.php (ancienne adresse)
├── confidentialite.html    Mentions légales et politique de confidentialité
├── products.json           Catalogue : 12 produits + fiche technique
├── .htaccess               Cache, en-têtes de sécurité, blocage des .md et .bat
├── verifier.bat            Vérification syntaxique de tous les fichiers PHP (double-clic)
├── favicon.svg, logo.svg   Identité visuelle
├── css/
│   ├── brand.css           Couleurs de marque communes site + panel, spinner
│   └── style.css           Styles du site public
├── js/
│   ├── script.js           Menu mobile, catalogue, fiche produit, panier, contact
│   └── client-auth.js      Fenêtre de connexion / inscription / mot de passe oublié
├── images/                 Photos des produits et de la bannière (13 JPG)
├── client/                 API JSON de l'espace client (appelée en fetch)
│   ├── session.php         État de session + jeton CSRF
│   ├── login.php           Connexion unique (admin puis client)
│   ├── logout.php          Déconnexion
│   ├── register.php        Inscription client
│   ├── forgot-password.php Envoi du code de réinitialisation
│   └── reset-password.php  Vérification du code + nouveau mot de passe
├── inc/
│   └── client_auth.php     Fonctions de l'espace client (comptes, session, anti-brute-force)
├── admin/
│   ├── login.php           Redirige vers la fenêtre de connexion du site
│   ├── logout.php          Déconnexion admin
│   ├── index.php           Redirige vers panel.php ou login.php
│   ├── panel.php           Tableau de bord
│   ├── config-email.php    Configuration des e-mails + logs d'envoi
│   ├── password.php        Changement du mot de passe admin
│   ├── users.php           Changement de l'adresse e-mail admin
│   ├── maintenance.php     Système, Journaux, Sauvegarde, Purger storage/tmp
│   ├── assets/admin.css    Styles du panel
│   └── inc/
│       ├── auth.php        Socle : JSON, sessions, CSRF, anti-brute-force, audit
│       ├── mail.php        Envoi des e-mails (PHPMailer), logs de débogage
│       └── layout.php      Gabarit commun du panel (en-tête, menu, pied)
├── lib/phpmailer/          PHPMailer (Exception.php, PHPMailer.php, SMTP.php)
├── data/                   Données JSON (accès HTTP interdit par .htaccess)
└── storage/
    ├── logs/               Logs d'envoi des mails (accès HTTP interdit)
    └── tmp/                Fichiers temporaires (accès HTTP interdit)
```

Un fichier `CLAUDE.md` (notes de contexte pour l'agent IA) se trouve aussi à la racine.
Les livrables du TP 2bis (`DOCUMENTATION.md`, `TODO_LIST.md`) sont rangés dans le dossier `Docs TP2 bis/`.

---

## 4. Points d'entrée et routage

Le site **n'a pas de routeur**. Chaque URL correspond directement à un fichier servi par Apache.

| Type d'URL | Traitement |
|---|---|
| `*.html` | Fichier statique. Le contenu dynamique est ajouté par JavaScript. |
| `contact.php` | Page PHP : affiche le formulaire, traite le `POST`, envoie le mail. |
| `client/*.php` | API JSON. Méthode `POST` obligatoire (`ot_api_post_only`) et jeton CSRF (`ot_api_guard_csrf`), sauf `session.php` (`GET`). |
| `admin/*.php` | Pages du panel. Chacune commence par `if (!ot_logged_in())` et redirige sinon vers `login.php`. |
| `admin/login.php` | Redirige vers `../index.html#connexion` ; le JavaScript ouvre alors la fenêtre de connexion. |
| `data/`, `storage/` | Refusés par Apache (`Require all denied` dans leur `.htaccess`). |
| `admin/inc/*.php`, `inc/*.php` | Réponse 403 : chaque fichier vérifie s'il est appelé directement. |

Exemple de parcours — connexion :
1. `index.html` charge `js/client-auth.js`, qui appelle `client/session.php` (jeton CSRF).
2. Le formulaire envoie un `fetch` vers `client/login.php`.
3. La réponse JSON indique `role: "admin"` (redirection vers `admin/panel.php`) ou `role: "client"` (on reste sur le site).

---

## 5. Stockage des données

Toutes les données sont dans des **fichiers JSON** du dossier `data/`. Il n'y a pas de base de données.

Lecture et écriture : `ot_read_json()` et `ot_write_json()` dans `admin/inc/auth.php`.
L'écriture passe par un fichier `.tmp` au nom unique puis un renommage, pour ne jamais laisser un fichier à moitié écrit.
Les mises à jour fréquentes (journal, messages, compteurs) passent par `ot_json_update()`, sous verrou (`<fichier>.lock`), pour qu'aucune donnée ne soit perdue en cas de requêtes simultanées.

| Fichier | Contenu | Champs principaux |
|---|---|---|
| `data/admin.json` | Comptes administrateurs | `id`, `username`, `email`, `nom`, `role`, `password_hash`, `cree_le`, `derniere_connexion`, `derniere_ip` |
| `data/clients.json` | Comptes clients | `id`, `nom`, `email`, `password_hash`, `cree_le`, `derniere_connexion`, `derniere_ip`, `actif` |
| `data/security.json` | Anti-brute-force admin, par IP | `attempts`, `next_attempt`, `blocked_until`, `last_seen` |
| `data/client_security.json` | Anti-brute-force client, par IP + délais par IP (`register`, `contact`) | mêmes champs ; `register` / `contact` : IP → heure de la dernière action |
| `data/audit.json` | Journal de connexion (500 derniers évènements) | `ts`, `datetime`, `event`, `ip`, `detail` |
| `data/contact_messages.json` | Messages du formulaire de contact (6 mois maximum, 500 au plus) | `datetime`, `nom`, `email`, `message`, `ip`, `mail_ok` |
| `data/email_config.json` | Configuration d'envoi des e-mails | `engine`, `from_name`, `from_email`, `to_email`, `smtp_host`, `smtp_port`, `smtp_secure`, `smtp_user`, `smtp_pass`, `debug_log` |
| `data/email_config.mailjet.json` | Ancienne configuration Mailjet conservée | — |
| `data/email_config.example.json` | Modèle de configuration **sans mot de passe** (non lu par le site) | mêmes champs que `email_config.json` |
| `products.json` (racine) | Catalogue public (lu par le navigateur) | `id`, `nom`, `categorie`, `prix`, `ancienPrix`, `description`, `image`, `note`, `avis`, `badge`, `caracteristiques`, `garantie` |

Données côté navigateur :
- **Panier** : `localStorage`, clé `omnitech_cart` (`js/script.js`). Il n'est pas envoyé au serveur.
- Les commandes sont **simulées** : aucune n'est enregistrée et aucun paiement n'existe.

Fichiers hors JSON :
- `storage/logs/mail.log` : logs d'envoi des e-mails (si l'option est cochée), rotation à 512 Ko (`admin/inc/mail.php`).

---

## 6. Authentification, sessions et sécurité

### Comptes et mots de passe
- Hachage **bcrypt, coût 12** (`ot_hash_password`, constante `OT_BCRYPT_COST`). Re-hachage automatique si le format change (`password_needs_rehash`).
- Règles client : 8 caractères, majuscule, minuscule, chiffre, caractère spécial (`ot_password_errors`).
- Règles admin : **20 caractères** minimum + mêmes règles (`ot_admin_password_errors`), y compris lors d'une réinitialisation.

### Connexion unique (`client/login.php`)
- Recherche d'abord un **administrateur** (identifiant ou e-mail), puis un **client** (e-mail).
- Message d'échec identique dans tous les cas : on ne peut pas savoir si un compte existe.
- Compte inconnu : vérification fictive sur `OT_DUMMY_HASH`, pour un temps de réponse identique (environ 1,16 s mesuré dans les deux cas).

### Anti-brute-force (`admin/inc/auth.php`)
- 3 s d'attente après chaque échec, délai croissant au-delà de 5 échecs (5 s → 5 min).
- Blocage de l'adresse IP pendant **2 heures** au-delà de 10 échecs.
- Deux compteurs séparés : `security.json` (identifiant admin) et `client_security.json` (autres).

### Sessions
- Session PHP native. Cookie `HttpOnly` et `SameSite=Lax` (`ot_session_start`).
- Régénération de l'identifiant à la connexion (`session_regenerate_id(true)`).
- Session admin fermée après **30 minutes d'inactivité** (`ADMIN_IDLE_TIMEOUT`, vérifié dans `ot_logged_in`). Le panel se recharge seul au bout de ce délai.

### Mot de passe oublié (`client/forgot-password.php`, `client/reset-password.php`)
- Code à **8 chiffres**, valable **5 minutes**, **5 essais** maximum, stocké **haché** en session.
- Le code n'est jamais affiché à l'écran. Une adresse inconnue reçoit la même réponse (demande fictive).
- La réponse part **avant** l'envoi de l'e-mail, après un délai fixe : le temps de réponse est identique pour un compte existant ou inconnu.

### Protection des formulaires
- Jeton **CSRF** sur tous les formulaires du panel et sur l'API `client/` (`ot_csrf_token`, `ot_csrf_check`, comparaison `hash_equals`).
- Contact : champ piège anti-robot `site_web`, 30 s minimum entre deux envois **par adresse IP**, validation côté serveur.
- Inscription : même champ piège et même délai de 30 s par IP (fonction commune `ot_rate_wait`).
- Refus et robots piégés enregistrés dans le journal (`contact_wait`, `contact_honeypot`, `client_register_wait`, `client_register_honeypot`).
- Échappement des sorties HTML : `ot_h()` (PHP) et `escapeHtml()` / `esc()` (JavaScript).

### Journal d'audit
- Connexions réussies et échouées, blocages, réinitialisations, actions de maintenance, expirations de session (`ot_audit_log`).
- Inscriptions refusées : délai de 30 s non respecté (`client_register_wait`) et robots piégés (`client_register_honeypot`). Elles sont classées dans le filtre « Échecs / blocages » (`admin/maintenance.php`).
- Consultable et filtrable dans Maintenance → Journaux.

---

## 7. Front-end

- **HTML** : 3 pages publiques (`index.html`, `produits.html`, `contact.php`) avec le même en-tête et le même pied de page, copiés dans chaque fichier.
- **CSS** : `css/brand.css` définit les couleurs de marque (`--brand-blue`, `--brand-violet`, `--brand-cyan`, dégradé). `css/style.css` (site) et `admin/assets/admin.css` (panel) s'appuient dessus.
- **Thème** : site clair avec en-tête et bannière sombres ; panel uniquement sombre. **Pas de bascule clair/sombre.**
- **Responsive** : 9 points de rupture (`@media`) dans `css/style.css`. Menu « hamburger » sur mobile. La fiche produit devient un panneau qui monte depuis le bas.
- **En-tête sur mobile** (tâche F5) : sous 720 px, le logo reste à gauche et les boutons « Connexion », panier et menu sont groupés à droite, en boutons de 42 px faciles à toucher. Sous 420 px, le logo n'affiche plus que son icône (le nom reste lu par les lecteurs d'écran) pour que « Connexion » garde son texte. Le menu du compte connecté est calé sur les bords de l'écran. Vérifié de 320 à 720 px sur les 4 pages : rien ne déborde ni ne se chevauche. Sur ordinateur, les liens « Accueil / Nos Produits / Contact » sont centrés sur la page (à partir de 1000 px) et le panier est groupé avec « Connexion » à droite ; entre 721 et 900 px, les espacements sont resserrés pour que « Nos Produits » reste sur une ligne.
- **Panel sur mobile** (tâche F2) : sous 640 px, chaque ligne des tableaux (journaux, messages de contact, fichiers, sauvegarde) s'affiche en carte, avec le libellé de chaque valeur à gauche. Plus aucun défilement horizontal à 375 px. Les libellés (`data-label`) sont recopiés depuis les titres de colonnes par le script commun de `admin/inc/layout.php` ; un nouveau tableau `.data-table` en profite donc sans modification. L'affichage sur ordinateur est inchangé.
- **JavaScript** (`js/script.js`, `js/client-auth.js`) :
  - chargement du catalogue en AJAX (`fetch("products.json")`), squelettes de chargement, filtres en temps réel ;
  - fiche produit, panier, validation du formulaire de contact ;
  - fenêtre de connexion accessible au clavier (focus piégé, touche Échap) ;
  - indicateur de chargement sur les boutons (`.is-loading`) ;
  - états vides illustrés (panier vide, recherche sans résultat).
- **Cache** : les balises `<script>` et `<link>` portent un numéro de version (`?v=…`) et le `.htaccess` impose la revalidation.

---

## 8. Intégrations externes

| Intégration | Utilisation | Fichiers |
|---|---|---|
| **Envoi d'e-mails** (PHPMailer) | Mails de contact, codes de réinitialisation, e-mail de test | `admin/inc/mail.php` |
| Moteurs disponibles | `php` (fonction `mail()`), `mailjet` (SMTP Mailjet), `smtp` (serveur SMTP classique) | `OT_MAIL_ENGINES` |
| Moteur actif | `smtp` — serveur, port et identifiants enregistrés dans `data/email_config.json` (non reproduits ici) | `data/email_config.json` |

Aucune autre API ni service tiers : pas de CDN, pas de police en ligne, pas de statistiques, pas de passerelle de paiement.

---

## 9. Configuration

| Où | Quoi |
|---|---|
| `admin/inc/auth.php` (constantes) | Chemins (`OT_ROOT`, `OT_DATA`, `OT_STORAGE`, `OT_TMP`…), `OT_BCRYPT_COST = 12`, `BF_DELAY_STANDARD = 3`, `BF_EXP_LIMIT = 5`, `BF_BLOCK_AFTER = 10`, `BF_BLOCK_SECONDS = 7200`, `RESET_CODE_TTL = 300`, `RESET_MAX_TRIES = 5`, `ADMIN_IDLE_TIMEOUT = 1800` |
| `admin/inc/mail.php` | `OT_MAIL_ENGINES`, `OT_MAIL_LOG`, `OT_MAIL_LOG_MAX` (512 Ko) |
| `js/script.js` | `CART_KEY`, `FREE_SHIPPING = 80` (livraison offerte) |
| `data/email_config.json` | Configuration des e-mails, modifiable dans Panel → Configuration des e-mails |
| `.htaccess` (racine, `data/`, `storage/`) | Cache navigateur ; interdiction d'accès aux données |
| Apache (`httpd-vhosts.conf`) | Hôte virtuel `omnitech.local` (hors du dossier du site) |

Il n'y a **pas de fichier `.env`** ni de configuration par environnement.

---

## 10. Limites connues et risques

Classées de la plus importante à la moins importante.

### Sécurité
1. ~~**`CLAUDE.md` est lisible par HTTP**~~ — **Corrigé (tâche S1)** : le `.htaccess` racine refuse désormais tous les fichiers `.md` (réponse 403).
2. **Mot de passe SMTP en clair** dans `data/email_config.json`. Le dossier est protégé par `.htaccess`. **Atténué (tâche D1)** : le `.gitignore` exclut les données réelles d'un dépôt Git, et `data/email_config.example.json` fournit un modèle sans mot de passe. Le mot de passe reste en clair sur le disque local, ce qui est nécessaire pour l'envoi SMTP.
3. ~~**Fuite par le temps de réponse dans « Mot de passe oublié »**~~ — **Corrigé (tâche S2)** : `client/forgot-password.php` répond après le même délai fixe dans tous les cas (environ 0,36 s), puis envoie l'e-mail une fois la réponse partie (`ot_api_json_then_continue` dans `inc/client_auth.php`).
4. **L'inscription révèle les comptes existants** : « Un compte existe déjà avec cette adresse e-mail » (`client/register.php`).
5. ~~**Aucune limite sur l'inscription**~~ — **Corrigé (tâche S3)** : `client/register.php` ignore les envois dont le champ piège `site_web` est rempli, et impose 30 s entre deux inscriptions depuis une même IP (`ot_client_register_wait`, mémorisé dans `client_security.json`, donc insensible à la suppression des cookies).
6. ~~**Limite du formulaire de contact contournable**~~ — **Corrigé (tâche S4)** : les 30 s entre deux messages sont mémorisées par adresse IP côté serveur (`ot_rate_wait` / `ot_rate_mark` dans `admin/inc/auth.php`, fichier `client_security.json`). Supprimer ses cookies ne permet plus de les contourner.
7. ~~**Pas d'en-têtes de sécurité HTTP**~~ — **Corrigé (tâche S5)** : le `.htaccess` racine envoie `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: same-origin` et `Permissions-Policy` sur toutes les réponses. Reste optionnel : une `Content-Security-Policy` (tâche S7).
   - *Limite restante* : l'en-tête `Server` annonce les versions d'Apache et de PHP (`Apache/2.4.65 (Win64) PHP/8.3.28`). Ce réglage (`ServerTokens Prod`, `expose_php = Off`) se fait dans `httpd.conf` et `php.ini` de WAMP, pas dans un `.htaccess`.
8. ~~**`lib/phpmailer/` est accessible par URL**~~ — **Corrigé (tâche S1)** : `lib/.htaccess` interdit le dossier (réponse 403) ; PHP continue de charger PHPMailer.

### Stockage des données
9. ~~**Écritures concurrentes**~~ — **Corrigé (tâche D2)** : le journal, les messages de contact, les compteurs anti-brute-force et les délais par IP passent par `ot_json_update()`, qui verrouille (`flock` sur un fichier `.lock`) tout le cycle *lire → modifier → écrire*. Test avec 10 processus simultanés : 500 évènements sur 500 (contre 22 sur 500 avant). Chaque écriture utilise aussi un fichier temporaire au nom unique.
   - *Limite restante* : les écritures complètes (configuration mail, comptes admin et clients, purges) ne sont pas verrouillées. Elles sont rares et faites par une seule personne.
10. **Mot de passe SMTP inclus dans les sauvegardes manuelles** du dossier `data/` (voir point 2). Les dépôts Git en sont protégés par le `.gitignore` (tâche D1), mais pas une simple copie du dossier.

### Qualité du code
11. ~~**Code mort**~~ — **Corrigé (tâche Q1)** : `ot_user_add`, `ot_user_delete` et `ot_client_find_by_id` ont été supprimées. Une recherche automatique confirme qu'aucune autre fonction PHP n'est inutilisée.
12. ~~**Doublon de contenu**~~ — **Corrigé (tâche Q2)** : l'accueil ne contient plus que la liste des produits mis en avant (`data-ids="1,3,4"` sur `#featured-grid`). Les cartes sont créées depuis `products.json` par la même fonction que le catalogue (`buildProductCard` dans `js/script.js`).
13. **En-tête et pied de page copiés** dans chaque page (`index.html`, `produits.html`, `contact.php`). Une modification du menu doit être faite trois fois.
14. **Numéro de version des fichiers CSS/JS à changer à la main** (`?v=…`) après chaque modification.
15. **Pas de tests fonctionnels automatisés.** La syntaxe se vérifie désormais d'un double-clic avec `verifier.bat` (tâche Q3 : `php -l` sur les 19 fichiers PHP, avec la version de PHP chargée par Apache). Le fonctionnement reste testé à la main dans le navigateur.

### Fonctionnel et conformité (RGPD)
16. ~~**Aucune mention d'information RGPD**~~ — **Corrigé (tâche R1)** : une mention sous le formulaire de contact (`contact.php`) et sous l'inscription (`js/client-auth.js`) indique les données conservées, leur usage et leur durée. Une page `confidentialite.html` (tâche R2), liée depuis le pied de page, détaille l'éditeur, l'hébergement local, les données, les durées de conservation, les cookies et les droits. Les messages de contact sont supprimés automatiquement après **6 mois** (tâche R3, `CONTACT_RETENTION_DAYS`, `ot_contact_purge_old`).
17. **Le client ne peut ni modifier ni supprimer son compte** : il n'existe pas de page « Mon compte ».
18. ~~**Newsletter factice présentée comme réelle**~~ — **Corrigé (tâche F1)** : la newsletter reste une démonstration (rien n'est enregistré), mais elle l'annonce clairement, avant l'envoi (« offre fictive, aucune adresse n'est enregistrée ») et après.
19. ~~**Sauvegarde (Maintenance)** : rubrique « API » erronée~~ — **Corrigé (tâche D3)** : l'export indique le moteur réellement actif (`moteur`, `fournisseur`, `identifiant`) ; le secret reste `[retiré]`.

---

## 11. Points forts

- **Sécurité des comptes soignée pour un projet de 1re année** : bcrypt coût 12, anti-brute-force progressif avec blocage IP, CSRF partout, temps de réponse constant à la connexion, expiration de session admin.
- **Aucune fuite d'existence de compte** à la connexion et lors de la demande de code.
- **Code de réinitialisation robuste** : haché, limité dans le temps et en nombre d'essais, jamais affiché.
- **Données protégées** : `data/`, `storage/`, `lib/` et les fichiers `.md` refusés par Apache, fichiers `inc/` non exécutables directement.
- **Logs d'envoi des mails sans secrets** : le mot de passe SMTP et le contenu des messages (codes) sont masqués (`ot_mail_clean_transcript`).
- **Aucune dépendance externe** hors PHPMailer : pas de CDN, pas de framework, pas de Composer.
- **Écriture JSON atomique** (fichier temporaire puis renommage).
- **Accessibilité** : fenêtres pilotables au clavier, attributs ARIA, messages `aria-live`.
- **Traçabilité** : journal d'audit filtrable, journal des messages de contact, purge confirmée par fenêtre.
- **Paiement 100 % simulé** : aucune donnée bancaire n'est demandée ni stockée.
