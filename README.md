# Helicaps : boutique en ligne

Le cahier des charges est dans `CLAUDE.md`, le design dans `design/`.

## Lancer le site sur cet ordinateur

```bash
npm install
npm run dev
```

Puis ouvrir http://localhost:3000.

Sans compte Supabase ni Stripe, le site tourne en **mode démo** : les données sont dans `.data/demo.json`
(supprimer ce fichier remet la boutique à zéro) et le paiement est simulé.

## Les deux côtés du site

| | Adresse | Qui y a accès |
|---|---|---|
| Boutique | `/` | Tout le monde. Aucun lien ne mène à l'admin. |
| Admin | `/admin` | Uniquement le compte dont l'e-mail est `ADMIN_EMAIL`. |

En mode démo, l'e-mail et le mot de passe de l'admin sont dans `.env.development.local` (`ADMIN_EMAIL`, `ADMIN_PASSWORD`).

Dans l'admin : onglet **Commandes** (à expédier / expédiées, « Marquer comme expédiée ») et onglet **Produits**
(stock −/+, interrupteur « En ligne », « Ajouter une casquette », « Modifier »).

Chaque page et chaque action de l'admin revérifie le compte côté serveur (`requireAdmin` dans `src/lib/auth.ts`).

## Mettre en ligne

Le pas-à-pas complet, écrit pour le propriétaire, est dans `Guide-mise-en-ligne-Helicaps.pdf` (source : `docs/guide/guide.html`).

En résumé : le code est déposé dans un dépôt GitHub privé du propriétaire, Netlify le construit et l'héberge
(`netlify.toml`), et chaque modification poussée sur `main` est mise en ligne automatiquement.
Les clés se saisissent dans Netlify (modèle : `.env.example`), jamais dans le code.

- **Supabase** : coller `supabase/schema.sql` dans SQL Editor, fermer les inscriptions, créer le compte propriétaire.
- **Stripe** : clé secrète, puis webhook `https://<domaine>/api/stripe/webhook` au format « Instantané » (snapshot)
  avec `checkout.session.completed` et `checkout.session.expired`.
- **Mondial Relay** : code enseigne dans `NEXT_PUBLIC_MONDIAL_RELAY_BRAND` (sinon le widget tourne avec le code de test).
- **Resend** : `RESEND_API_KEY` et `EMAIL_FROM` (domaine vérifié).

`helicaps-site.zip` est le paquet à envoyer au propriétaire : le code sans les fichiers locaux. Pour le refaire : `npm run zip`.

Solution de secours sans GitHub, depuis ce Mac : `npx netlify-cli login`, `npx netlify-cli sites:create`, puis `npm run deploy`.

## À remplir par le propriétaire

- `src/lib/content.ts` : réseaux sociaux, avis clients (la section reste cachée tant que la liste est vide),
  textes « Livraison et retours », « Entretien », CGV, mentions légales, confidentialité, contact.
- Les vraies photos : dans l'admin, « Modifier » la casquette et remplacer les visuels provisoires.
  Une photo PNG détourée (fond transparent) se pose sur l'éclairage studio ; cocher « Photo portée » pour une photo plein cadre.

## Sécurité

Le code a été relu catégorie par catégorie du Top 10 de l'OWASP (édition 2025). Ce qui est en place :

| Catégorie | Dans ce projet |
|---|---|
| Contrôle d'accès | Chaque page et chaque action de `/admin` revérifie le compte (`src/lib/auth.ts`). Base fermée par RLS : les clés publiques ne lisent que le catalogue en ligne. Aucune adresse fournie par un visiteur n'est appelée par le serveur. |
| Configuration | En-têtes de sécurité sur toutes les réponses (`next.config.ts`) et Content-Security-Policy stricte à nonce (`src/proxy.ts`). Mode démo impossible en production. |
| Chaîne logicielle | `package-lock.json`, `npm audit` sans alerte. Le widget Mondial Relay et ses scripts de CDN sont enfermés dans une `<iframe sandbox>` (`src/app/relais/route.ts`), avec empreintes d'intégrité quand c'est possible. |
| Chiffrement | HTTPS imposé (HSTS). Mots de passe gérés par Supabase Auth. Cookies de session `HttpOnly`, `Secure`, `SameSite=Lax`. Aucun secret dans le code. |
| Injection | Requêtes paramétrées uniquement. Sorties échappées par React, par `esc()` dans les e-mails, et dans le JSON-LD. Photos vérifiées sur leur contenu réel, pas sur leur nom. |
| Conception | Prix, stock, réduction et frais de port recalculés côté serveur. Réservation de stock atomique. Limites anti-abus par visiteur (connexion, paiement, code promo). Jamais de paiement simulé hors mode démo. |
| Authentification | Un seul compte admin, inscriptions fermées, 5 essais par quart d'heure, message d'erreur identique que l'e-mail existe ou non. |
| Intégrité | Signature des webhooks Stripe vérifiée, commande créée une seule fois même si l'événement est rejoué. |
| Journalisation | Journal de sécurité en JSON (`audit()` dans `src/lib/security.ts`) : connexions, refus, limites atteintes, actions de l'admin, erreurs de paiement. Visible dans Netlify > Logs > Functions. |
| Cas d'erreur | En cas de panne ou de configuration manquante, le site refuse plutôt que de laisser passer (voir les tests de `supabase/schema.test.mjs`). |

Ce que le code ne peut pas faire à la place du propriétaire : choisir un mot de passe long et unique, activer la double
authentification sur Supabase, Stripe, Netlify, GitHub et OVH, et ne jamais partager une clé secrète.
Il n'y a pas d'alerte automatique : le journal doit être consulté. Ce n'est pas un audit par un professionnel indépendant.

Une base créée avant le 13/10/2026 doit recevoir `supabase/mise-a-jour-securite.sql` (sinon les limites anti-abus restent inactives).
Pour tester une version de production en mode démo sur son propre ordinateur : `ALLOW_DEMO=1`.

## Vérifications

```bash
npm test
```

Teste le calcul des prix et rejoue `supabase/schema.sql` sur un Postgres embarqué : réservation de stock tout ou rien,
commande créée une seule fois, stock rendu si le paiement expire, et accès refusé avec les clés publiques.

## Ce qui n'a pas encore pu être testé

Faute de comptes, ces parties sont écrites mais n'ont jamais tourné contre le vrai service :
`src/lib/store/supabase.ts` (lecture/écriture Supabase), la connexion admin Supabase, la création de session
Stripe Checkout et son webhook, l'envoi d'e-mail Resend. À tester avec des clés de test avant toute mise en ligne.
