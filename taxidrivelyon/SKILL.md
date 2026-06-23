---
name: "taxidrivelyon"
description: "Maintenance et évolution du site Taxi Drive Lyon (taxidrivelyon.fr), un site de réservation taxi/VTC/chauffeur privé premium. Backend PHP, flux de réservation, dispatch SMS (Free Mobile) + Telegram, commissions PayPal.me, facturation, espace chauffeur, panel admin, déploiement Ionos. À utiliser dès qu'on travaille sur les fichiers du site Lyon (index.html, send_reservation.php, admin.php, facture.php, chauffeur.php, merci.php, commission.php) ou qu'on parle de réservations, dispatch, commissions, factures ou design premium du site."
license: MIT
metadata:
  version: 1.0.0
  author: mhkr69
  category: web
  updated: 2026-06-23
---

# Taxi Drive Lyon — Site de réservation premium

Tu es le développeur de référence du site **taxidrivelyon.fr**, un service de taxi / VTC / chauffeur privé haut de gamme à Lyon. Le site est en PHP + HTML/CSS/JS vanilla, hébergé chez **Ionos**, et déployé par **upload FTP manuel** des fichiers (il n'y a pas de pipeline CI/CD).

## Identité & règles non négociables

- **Branding** : noir & or premium (`--gold: #C9A96E` / variantes), typographies Cormorant Garamond + Inter + Syne. Thème clair beige possible. Logo texte « TDL — Taxi Drive Lyon ».
- **Numéro public partout** : `07 69 19 93 83` (href `tel:0769199383`). À afficher sur toutes les pages publiques.
- **Numéro interne SMS dispatch** : la ligne Free Mobile du patron (≠ numéro public). Ne jamais l'afficher publiquement.
- **JAMAIS mentionner Telegram sur les pages publiques.** Telegram sert uniquement au dispatch interne.
- **Mobile d'abord** : la majorité des réservations viennent du téléphone. Toute nouveauté doit rester rapide sur mobile — désactiver les effets lourds (blur, glows animés, parallax, 3D, flip) sous 680px.
- Paiement des commissions chauffeurs via **PayPal.me** (`paypal.me/mhkr6469/{montant}`), pas Stripe.

## Carte des fichiers

| Fichier | Rôle |
|---------|------|
| `index.html` | Site vitrine + formulaire de réservation (Google Maps autocomplete, estimation prix, carte de trajet, effets 3D desktop). Envoie en POST JSON vers `send_reservation.php`. |
| `send_reservation.php` | Cœur du backend. Reçoit la réservation, calcule prix/commission, sauvegarde la facture JSON dans `/factures/`, envoie emails + SMS (Free Mobile) + Telegram (groupe + privé). Renvoie `{reference, ...}`. |
| `merci.php` | Page de confirmation post-réservation (affiche la réf + conversion Google Ads). |
| `facture.php` | Récupération de facture par référence. Affiche la facture seulement si `statut === 'terminee'` (validée en admin). |
| `admin.php` | Panel admin (mot de passe). 2 onglets : Courses (valider → `terminee`) et Chauffeurs (accepter/refuser candidatures). |
| `chauffeur.php` | Formulaire « Devenir chauffeur » avec upload de documents (carte VTC, permis, CNI, photos). Sauve dans `/chauffeurs/`. |
| `commission.php` | Page de paiement d'une commission. |

## Flux de réservation (de bout en bout)

1. Client remplit le formulaire dans `index.html` (mode **Immédiat**, **Plus tard** ou **Mise à dispo**).
2. JS calcule km (arrondi 1 décimale) + durée via Google Distance Matrix, estime le prix, POST JSON → `send_reservation.php`.
3. PHP génère une **référence** `TDL-{date}-{hash}`, calcule la **commission** par tranche de km, sauvegarde `/factures/{ref}.json` avec `statut: en_attente`.
4. PHP envoie : email chauffeur, email client (si fourni), **SMS dispatch** (Free Mobile, vers la ligne du patron), **SMS confirmation client** à recopier, **Telegram groupe** (chauffeurs) + **Telegram privé** (patron, fiche complète + lien commission PayPal).
5. Le patron valide la course dans `admin.php` → `statut: terminee` → la facture devient accessible dans `facture.php`.

## Points de configuration (en haut de `send_reservation.php`)

- `$EMAIL` : contact@taxidrivelyon.fr
- `$FREE_LOGIN` / `$FREE_APIKEY` : identifiants API SMS Free Mobile (envoie **uniquement vers la propre ligne Free** du patron — ne peut PAS écrire au client).
- `$TG_TOKEN`, `$TG_CHAT_ID` (groupe), `$TG_OWNER_ID` (privé) : bot Telegram.
- `$PAYPAL_ME` : identifiant PayPal.me pour les liens de commission.
- `$TRANCHES_COMMISSION` : grille commission par distance (0-2km→4€, … +30km→20€).

## Règles métier importantes

- **Course immédiate** = `date` vide. Toujours détecter l'immédiat avec `empty($date)`, **jamais** avec `$heure` (qui prend par défaut « Non précisée »).
- **Mise à disposition** : `destination` contient « disposition » ou `prix` contient « devis ». Dans ce cas → arrivée « À définir », commission « À voir avec le dispatcher », pas de calcul km.
- **km** : toujours arrondis à 1 décimale (JS `Math.round(d/100)/10` + PHP `round(...,1)`).
- **Google Maps** : biais sur Lyon via `bounds` (LatLngBounds autour de 45.6–45.9 / 4.7–5.0) + `componentRestrictions: {country:'fr'}`, `strictBounds:false`.
- **Limite SMS Free** : impossible d'envoyer automatiquement au client. Pour un vrai SMS client il faut un service payant (Brevo, SMSmode, OVH SMS…).

## Déploiement

Pas de CI/CD. Après modification, **lister explicitement les fichiers à uploader sur Ionos** (FTP) dans la racine du site. Les images (`lyon.jpg`, `berline.png`, `van.png`, `luxe.png`, `favicon.png`) vont aussi à la racine.

## Conseils d'intervention

- Avant d'éditer, repérer la section concernée par sa bannière de commentaire (`// ════ ... ════`).
- Pour toute requête « rendre plus premium » : privilégier des effets CSS légers (tilt 3D, glassmorphism, dégradés dorés, reveal au scroll) et **toujours les couper sous 680px**.
- Ne jamais casser le formulaire de réservation : c'est le seul canal de conversion.
- Vérifier la cohérence du numéro public `07 69 19 93 83` après chaque modif de texte.
- Toujours rappeler à l'utilisateur d'uploader les fichiers modifiés sur Ionos (il oublie souvent et teste l'ancienne version en cache → conseiller `Ctrl+Maj+R`).
