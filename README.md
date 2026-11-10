# EasyEat

**Application web gratuite qui aide les personnes cœliaques à manger sans gluten en sécurité** : photo d'un menu ou d'une étiquette → verdict par plat (sûr / à vérifier / à éviter) avec la question à poser au serveur, scan de code-barres avec alerte de rappel officiel, carte de restaurants sans gluten, recettes.

🔗 https://valentinhnn.github.io/EasyEat/

> *EN: Free web app helping people with coeliac disease eat gluten-free safely. Static front end on GitHub Pages, serverless back end on Cloudflare Workers, LLM-based menu and label analysis with a measured reliability test bench, and a sourced map of 1,000+ restaurants in 54 French cities.*

---

## Le problème

Pour une personne cœliaque, la moindre trace de gluten rend malade. Au restaurant, le gluten est souvent caché (viande farinée, sauce liée, friture partagée) et le personnel rarement formé. EasyEat donne un verdict rapide, explique pourquoi, et indique **la question précise à poser** quand un doute existe.

## Fonctionnalités

- **Menu** : analyse d'une photo de carte (ou de la carte en ligne d'un restaurant), verdict par plat, question au serveur traduite dans 16 langues, partage du résumé.
- **Produit** : scan de code-barres (Open Food Facts) avec vérification des rappels officiels (RappelConso), ou analyse d'une photo d'étiquette.
- **Restaurants** : carte Leaflet / OpenStreetMap de plus de 1 000 adresses dans 54 villes, filtres, avis communautaires modérés, favoris partageables, suggestions d'adresses.
- **Recettes** : génération sans gluten, liste de courses, fiche à envoyer à un hôte.
- **Voyage** : fiche cœliaque à montrer au serveur en 16 langues, avec conseils par pays.
- Thème clair / sombre, installable sur l'écran d'accueil (PWA), historique local, signalement des erreurs de verdict.

## Architecture

```
Navigateur (index.html, JS sans framework)
   │  GitHub Pages (statique)
   ▼
Cloudflare Worker (serverless)                 APIs publiques appelées par le navigateur
   ├─ proxy IA : cascade de modèles Gemini      ├─ Open Food Facts (produits)
   │   avec repli automatique (quota / panne /  ├─ RappelConso (rappels officiels)
   │   réponse invalide)                        └─ Overpass / OpenStreetMap (horaires)
   ├─ quotas et limites anti-abus
   ├─ avis, modération, suggestions, signalements
   └─ Cloudflare KV (stockage clé-valeur)
```

Le code du Worker n'est pas publié dans ce dépôt (il contient la logique de quota et les instructions données aux modèles) ; il est déployé avec Wrangler.

## Sécurité

- **Secrets côté serveur uniquement** (clé d'API IA, clé Turnstile) ; le navigateur ne parle jamais directement au fournisseur d'IA.
- **CORS strict** sur une liste d'origines, sans jamais renvoyer `Access-Control-Allow-Origin: null` (faille connue via iframe sandboxée).
- **Quotas par visiteur** stockés dans KV, avec l'IPv6 réduite à son préfixe /64 (sinon un abonné change d'adresse pour un quota illimité) et un anti-rafale en mémoire.
- **Budget d'écritures KV** (1 000/jour en gratuit) protégé : limites en mémoire pour les routes de lecture, Turnstile (anti-robot) sur les formulaires publics (avis, suggestions, signalements), statistiques agrégées et écrites par lots.
- **Fichiers envoyés** validés par leur signature binaire réelle (magic bytes), pas par le type déclaré.
- **Identité** : Google Sign-In, jeton vérifié côté serveur (audience, émetteur, e-mail vérifié) ; droits admin attribués côté serveur ; aucun mot de passe, aucun jeton stocké dans le navigateur.
- **Lecture de sites tiers (cartes en ligne)** : protection SSRF (schémas autorisés, pas d'IP brute ni de nom local, revérification après redirection, taille et délai limités) et protection contre l'injection d'instructions (texte tiers délimité et traité comme donnée par le modèle).
- **XSS** : toute donnée issue d'un visiteur ou du modèle est échappée avant insertion dans le DOM.
- **RGPD** : mentions légales et sous-traitants documentés, données minimales, statistiques anonymes.

## Fiabilité de l'IA

Un verdict faux peut rendre malade : la fiabilité est donc **mesurée**, pas supposée.

- Banc de tests avec verdicts attendus définis à l'avance : 60 plats pièges, puis plusieurs jeux de plats **inédits**, figés avant tout essai et jamais utilisés pour régler les instructions (pour éviter de « tricher » sur le jeu de test).
- Résultats : **0 verdict dangereux sur 120** sur les plats pièges ; **0 sur 216** sur 108 plats inédits avec le modèle actuel, garde-fous compris (une erreur de l'ancien modèle principal a conduit à ajouter un garde-fou déterministe) ; 26 étiquettes réelles sur 27 correctement classées.
- Garde-fous déterministes après la réponse du modèle (verdict inconnu ⇒ « à vérifier », plats presque toujours farinés jamais déclarés « sûr »), validation du JSON et repli sur un autre modèle.
- Comparaison avec une IA généraliste interrogée sans ces règles, et mesure des temps de réponse par modèle.
- Détails publics : [page Fiabilité](https://valentinhnn.github.io/EasyEat/fiabilite.html).

## Données restaurants

Collecte à partir de sources spécialisées (annuaires pour cœliaques, guides vérifiés, offices de tourisme), recoupement (« safe » = 100 % sans gluten confirmé par deux sources), géocodage via la Base Adresse Nationale, quartiers via OpenStreetMap, rapport sourcé pour chaque adresse.

## Démarche

Projet personnel que j'ai conçu, piloté, testé et mis en production : définition du produit et des priorités, choix d'architecture, méthode de test de l'IA, revue de sécurité, déploiement et suivi. Le code a été écrit avec l'aide d'un assistant IA (Claude), sous ma direction.

## Limites et suite

- L'IA peut se tromper : chaque verdict rappelle de confirmer auprès du restaurant, et un bouton permet de signaler une erreur.
- Prochaine étape : bêta avec des utilisateurs cœliaques, puis migration vers Cloudflare Pages pour ajouter des en-têtes de sécurité HTTP (impossibles sur GitHub Pages).

## Licence

Tous droits réservés (voir [LICENSE](LICENSE)). Le code est visible mais n'est pas open source.
