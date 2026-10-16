# Helicaps : boutique en ligne

Site e-commerce de **Helicaps**, marque de casquettes streetwear haut de gamme.
Accroche : **« Le luxe à tous les coins de rue. »**

Ce fichier est le cahier des charges du projet. Relis-le avant chaque tâche.
La direction visuelle validée est **B « Vitrine »** : luxe épuré, inspiré des règles Apple (Human Interface Guidelines).

---

## 1. Le produit

- 1 seul modèle de casquette, **3 coloris : noir, bleu, vert**.
- Taille unique **M**.
- Prix : **90 €** TTC.
- **3 pièces par coloris**, soit 9 au total. La rareté est l'argument de vente principal.
- Le modèle de données doit quand même permettre d'ajouter d'autres casquettes plus tard (l'admin a un formulaire « ajouter une casquette »).

---

## 2. Stack technique (proposition, à valider avec moi avant d'installer)

- **Next.js** (App Router) + **TypeScript**.
- **Tailwind CSS v4**, thème branché sur `design/tokens.css` (les tokens sont la source de vérité, pas de valeurs en dur).
- **Supabase** : base Postgres (produits, stock, commandes), Auth (connexion du propriétaire à l'admin), Storage (photos).
- **Stripe Checkout** (page de paiement hébergée par Stripe) : CB, Apple Pay, Google Pay. Webhooks pour créer la commande et décrémenter le stock.
- **Mondial Relay** : widget officiel de choix du point relais (vérifie la doc à jour). En V1 le propriétaire crée les étiquettes lui-même sur le site Mondial Relay ; l'API d'étiquettes, ce sera pour la V2.
- **E-mails** : reçus Stripe pour la confirmation ; un e-mail « Commande expédiée » via Resend (ou équivalent gratuit).
- **Hébergement** : selon la documentation de Vercel, le plan gratuit (Hobby) est réservé à un usage non commercial. Pour une boutique, il faut soit Vercel Pro, soit un hébergeur dont l'offre gratuite accepte le commercial. Signale-moi le choix avant de déployer.
- **Nom de domaine** : déjà chez OVH, au nom du propriétaire.

Budget très serré : rester sur des offres gratuites ou à l'usage partout où c'est possible.

---

## 3. Fichiers de design fournis

| Fichier | Contenu |
|---|---|
| `design/tokens.css` | Toutes les couleurs, typos, rayons, espacements, durées. **À importer tel quel.** |
| `design/maquettes/accueil-mobile.dc.html` | Maquette de l'accueil mobile (390 px), validée. |
| `design/maquettes/accueil-desktop.dc.html` | Maquette de l'accueil desktop (1440 px, fluide), validée. |
| `design/maquettes/*.png` | (si présents) captures des maquettes exportées depuis l'outil de design. |
| `public/images/placeholders/cap-*.svg` | Visuels **provisoires** des 3 coloris, remplacés plus tard par les vraies photos. |

Les `.dc.html` sont au format de l'outil de maquette : ne pas les exécuter.
- `{{ variable }}` = valeur calculée dans le bloc `<script type="text/x-dc">` en bas du fichier.
- `<sc-for>` = boucle, `onClick="{{ fn }}"` = gestionnaire d'événement.
- Les styles sont en ligne : reprends-en **les valeurs exactes** (tailles, espacements, rayons, textes, SVG des icônes), mais réécris-les proprement en composants + tokens.

Les pages produit, panier et admin n'ont pas encore de maquette : construis-les avec les règles des sections 4 et 5, dans le même vocabulaire visuel que l'accueil.

---

## 4. Règles de design (direction B)

### Ambiance
- **Noir dominant** (`--color-bg`), surfaces carbone (`--color-surface`).
- **Orange uniquement** sur l'action principale de l'écran et sur les infos de rareté. Une seule action orange par écran.
- **Texte noir sur l'orange, toujours.** Le blanc sur orange est interdit (contraste 3,1:1).
- Photos produit posées sur un **éclairage studio** (`--stage-hero`, `--stage-card`), jamais sur un aplat.
- Composition centrée sur le hero, grandes respirations, coins arrondis (cartes 24–32 px, boutons en pilule).

### Typographie
- **Geist** seule (Google Fonts / `next/font`), poids 400, 500 et 600.
- Échelle dans `tokens.css` : display, title, card-title, body (17 px), caption (15 px), footnote (13 px). **Rien sous 13 px.**
- Logotype = texte « HELICAPS » en Geist 600, interlettrage 0,34 em, toujours blanc. Pas de logo image.

### Accessibilité et règles Apple (non négociables)
- Zone tactile **≥ 44 × 44 px** pour tout élément cliquable (icônes, pastilles, flèches, boutons d'admin).
- Boutons pilule de 50 px (mobile) / 52 px (desktop) de haut.
- Contraste texte ≥ 4,5:1 (les tokens sont déjà vérifiés, ne pas en inventer d'autres).
- Vrais éléments HTML : `<button>`, `<a href>`, `<input>` + `<label>`, `<dialog>`. Jamais de `div` cliquable.
- `aria-label` sur chaque bouton icône ; `aria-pressed` sur les pastilles de coloris.
- Focus clavier visible (`--focus-ring`), navigation complète au clavier.
- `prefers-reduced-motion` respecté (déjà géré dans les tokens).
- Sur mobile, l'action principale est en bas de l'écran, à portée de pouce (avec `env(safe-area-inset-bottom)`).

### Mouvement
- Changement de coloris : fondu de la photo en 200 ms.
- Tiroir panier : glisse depuis la droite en 320 ms (`--ease-out`).
- Ajout au panier : ouverture du tiroir + compteur du panier qui s'incrémente. Pas de pop-up.
- Aucune animation en boucle, aucun effet d'apparition au scroll sur chaque section.

### Rareté (composant partagé)
- Indicateur : 3 segments arrondis (4 px de haut), orange = disponible, `--color-inactive` = vendu.
- Texte toujours présent à côté (jamais la couleur seule) :
  - 3 ou 2 restantes → « Plus que N pièces »
  - 1 → « Dernière pièce »
  - 0 → « Épuisé » (texte gris, bouton désactivé `--color-disabled-bg`, pastille barrée)
- Sur l'accueil : « X pièces disponibles sur 9 », calculé depuis la base.
- Le stock affiché vient **toujours** de la base de données, jamais codé en dur.

### Ton des textes
- Français, sobre, vouvoiement. Boutons à l'infinitif ou à la 1re personne (« Choisir mon coloris », « Ajouter au panier »).
- Pas de faux contenu : les avis clients et les liens réseaux sont des **emplacements à remplir** (`[@helicaps]`, `[Texte d'un vrai avis]`). Tant qu'il n'y a pas de vrais avis, la section avis est masquée.

---

## 5. Écrans

### 5.1 Accueil `/` (maquettes fournies)
1. Barre de navigation collante, 52 px, fond `--color-nav-bg` + `backdrop-filter: saturate(180%) blur(20px)`. Mobile : menu, logotype centré, panier. Desktop : liens (Coloris, Avis, Livraison), logotype centré, Instagram, TikTok, panier.
2. Hero centré : grand titre, sous-titre « Une casquette. Trois coloris. Neuf pièces. », bouton « Choisir mon coloris », « 90 €, taille unique M ».
3. Scène : les 3 casquettes ensemble sur éclairage studio + « X pièces disponibles sur 9 ».
4. Bloc coloris interactif : grande carte photo, pastilles noir/bleu/vert, stock, bouton « Voir la casquette bleue » (accordé : noire, bleue, verte).
5. Réassurance (4 tuiles) : point relais Mondial Relay 3 €, livraison offerte dès 180 €, paiement sécurisé, expédiée sous 2 jours.
6. Avis clients (carrousel horizontal), masqué s'il n'y a pas d'avis.
7. Réseaux : liste groupée Instagram / TikTok.
8. Footer : logotype, Livraison et retours, Contact, CGV, Mentions légales, ©.

### 5.2 Page produit `/casquette/[coloris]`
- **Galerie swipeable** (scroll-snap, points de pagination, flèches ≥ 44 px sur desktop) : photos studio + photos portées du coloris choisi.
- **Pastilles de coloris** : changent la galerie (fondu 200 ms) et l'URL, sans rechargement.
- Nom, prix 90 €, puce non cliquable « Taille unique M ».
- Rareté (composant partagé) : « Plus que 2 pièces ».
- Description + accordéons « Livraison et retours », « Entretien » (contenu à fournir : mettre des emplacements).
- **Mobile : barre fixe en bas** avec prix + bouton pilule « Ajouter au panier » (safe-area). Désactivé « Épuisé » à 0.
- Quantité max = stock restant.
- Desktop : 2 colonnes (galerie à gauche, infos collantes à droite).
- Données structurées `Product` (prix, disponibilité) pour le SEO.

### 5.3 Panier en tiroir latéral
- `<dialog>` modal qui glisse depuis la droite : plein écran sur mobile, 420 px sur desktop, voile `--color-overlay`. Focus piégé, Échap ferme, retour du focus au bouton panier.
- En-tête : « Panier (n) » + bouton fermer (44 px).
- Lignes : vignette (rayon 12), « Casquette Helicaps, Bleu », « Taille M », sélecteur de quantité −/+ (44 px, borné au stock), prix, « Retirer ».
- **Livraison offerte** : « Plus que X € pour la livraison offerte » + barre de progression orange ; « Livraison offerte » quand le panier atteint 180 €.
- **Code promo** : champ + « Appliquer », états succès / erreur clairs (ex. « Ce code n'existe pas ou a expiré »). Codes gérés dans Stripe (Promotion Codes), validés côté serveur.
- **Point relais** : ligne « Choisir un point relais » → widget Mondial Relay (code postal, carte/liste). Une fois choisi : nom + adresse + « Modifier ». Le paiement est bloqué tant qu'aucun point n'est choisi.
- Récapitulatif : sous-total, livraison (3 € ou Offerte), réduction, **total**.
- Bouton pilule orange « Payer 93,00 € » → Stripe Checkout.
- Panier vide : « Votre panier est vide » + bouton « Voir les coloris ».
- Le panier est conservé côté navigateur (localStorage), mais **prix, stock et frais de port sont toujours recalculés côté serveur** au paiement.

### 5.4 Admin mobile `/admin` (propriétaire non technique)
Pensé d'abord pour un téléphone : gros boutons, peu de texte, pas de jargon.
- Connexion par e-mail (Supabase Auth), un seul compte propriétaire. Toutes les routes `/admin` protégées.
- **Barre d'onglets en bas** : Commandes, Produits.
- **Commandes** : deux listes « À expédier » / « Expédiées ». Chaque carte : n° de commande, date, nom du client, articles (coloris × quantité), total, **point relais** (nom, adresse, identifiant Mondial Relay, bouton « Copier »).
  Bouton principal **« Marquer comme expédiée »** → confirmation, champ facultatif « Numéro de suivi », puis e-mail automatique au client.
- **Produits** : liste avec photo, nom, prix, stock par coloris (boutons −/+ de 44 px), interrupteur « En ligne ».
- **Ajouter une casquette** : photos (depuis la pellicule du téléphone, réordonnables, une photo principale par coloris), nom, prix (€, clavier numérique), description, coloris (nom + couleur de la pastille + stock + photos), bouton « Publier ». Champs de 50 px de haut, libellés au-dessus, messages d'erreur sous le champ concerné.

### 5.5 Pages légales
CGV, mentions légales, livraison et retours (droit de rétractation de 14 jours), politique de confidentialité. Le contenu sera fourni par le propriétaire : crée les pages avec des emplacements clairement marqués.

---

## 6. Données et règles métier

Tables (à adapter, toutes avec RLS) :
- `products` (id, name, slug, description, price_cents, is_active, created_at)
- `variants` (id, product_id, color_name, color_slug, swatch_hex, stock, sort_order)
- `product_images` (id, variant_id, url, alt, kind `studio` | `worn`, sort_order)
- `orders` (id, number, stripe_session_id, email, customer_name, phone, relay_id, relay_name, relay_address, relay_zip, relay_city, subtotal_cents, shipping_cents, discount_cents, total_cents, status `paid` | `shipped` | `cancelled`, tracking_number, shipped_at, created_at)
- `order_items` (order_id, variant_id, quantity, unit_price_cents)
- `stock_reservations` (stripe_session_id, variant_id, quantity, expires_at)

Lecture publique : produits, variantes et images actifs seulement. Commandes : admin uniquement.

Règles :
- **Livraison** : 3 € en point relais Mondial Relay, **offerte dès 180 €** (après réduction). Calcul côté serveur uniquement.
- **Stock = 3 pièces, donc zéro survente.** À la création de la session Stripe, réserver le stock (transaction atomique) pour la durée de vie de la session ; décrémenter définitivement sur `checkout.session.completed` ; libérer sur `checkout.session.expired`. Vérifier la signature des webhooks.
- Le point relais choisi passe dans les métadonnées de la session Stripe et est enregistré sur la commande.
- Expédition annoncée sous 2 jours.
- Montants stockés en centimes, affichés au format français (« 90 € », « 93,00 € »).

---

## 7. Qualité attendue
- Mobile d'abord, puis desktop. Tester à 375, 390, 768, 1280 et 1440 px.
- `next/image` (AVIF/WebP), image du hero prioritaire, polices via `next/font`.
- Lighthouse : viser ≥ 90 partout sur mobile.
- Métadonnées FR, image Open Graph, sitemap.
- Pas de clés secrètes dans le code : `.env.local` + `.env.example` documenté.

## 8. Façon de travailler
1. Avant de coder, propose-moi un plan par étapes et attends ma validation.
2. Une étape = une fonctionnalité testable (ordre conseillé : base du projet + tokens + nav/footer → accueil → page produit → panier → paiement Stripe + stock + Mondial Relay → admin → e-mails, pages légales, SEO).
3. Demande-moi les clés et comptes (Supabase, Stripe, Mondial Relay, Resend) au moment où tu en as besoin, pas avant.
4. Si une règle de ce fichier pose problème techniquement, dis-le et propose une alternative au lieu de la contourner.
