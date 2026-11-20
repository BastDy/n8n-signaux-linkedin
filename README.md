# Signaux LinkedIn Toronto

Un pipeline de veille qui repère chaque matin les posts LinkedIn où quelqu'un recrute en RevOps, Sales Ops ou CRM à Toronto (ou en remote au Canada), les classe par pertinence et les envoie dans un e-mail récapitulatif.

## Pourquoi

Les offres d'emploi publiées sur les job boards ne sont qu'une partie du marché. Beaucoup de recrutements commencent par un post : un fondateur qui cherche son premier profil RevOps, un VP Sales qui veut quelqu'un pour reprendre HubSpot. Ces posts disparaissent du fil en quelques heures.

Les outils de « signal-based selling » qui captent ce type de signal coûtent entre 159 $ et 2 500 $ par mois, ou ont fermé (Trigify, octobre 2026). Ce repo montre qu'on peut obtenir l'essentiel pour quelques dollars par mois avec des briques standard.

## Ce que fait le workflow

```
Cron 7 h 15 et 16 h (Toronto)
  ├─ LinkedIn (passage de 7 h 15 seulement)
  │    → Apify : 5 requêtes booléennes, dernières 24 h
  │    → Code : dédoublonnage (mémoire 30 jours) + scoring
  └─ X (aux deux passages)
       → Apify : tweets des comptes GTM suivis depuis le dernier passage
       → Code : retire retweets et réponses, dédoublonne, score
  → Un seul e-mail HTML (LinkedIn + X le matin, X seul l'après-midi)
```

Pas d'e-mail si rien de neuf (sauf erreur Apify, signalée dans l'e-mail).

### Comptes X suivis

La liste est en haut du nœud « Préparer le passage » (`COMPTES_X`), en handles sans @ ou en URL de profil. Les comptes sont groupés par 10 dans des requêtes `from:a OR from:b … since_time:… -filter:replies -filter:retweets`.

### Scoring X

| Critère | Points |
|---|---|
| Mot-clé métier (RevOps, GTM, HubSpot, Clay, outbound, pipeline…), max 3 | +20 chacun |
| Engagement comparé à la moyenne glissante du compte | jusqu'à +30 |
| Tweet de moins de 6 h | +10 |
| Lien, thread ou ressource (template, playbook…) | +10 |
| Signal de recrutement (« hiring », « join our team »…) | +25 |

Le scoring LinkedIn est volontairement simple et lisible :

| Critère | Points |
|---|---|
| Mot-clé métier (RevOps, Sales Ops, HubSpot, Salesforce, GTM…), max 3 | +20 chacun |
| Auteur décideur (fondateur, CEO, VP, Head of, Director…) | +25 |
| Toronto / GTA / Ontario | +15 |
| Remote + Canada | +10 |
| Invitation au contact direct (« DM me », « reach out »…) | +10 |
| Auteur recruteur ou agence | −10 |

## Stack et coût

| Brique | Rôle | Coût mensuel |
|---|---|---|
| n8n auto-hébergé (Docker) | Orchestration | 0 $ (serveur gratuit Oracle) à ~6 $ (VPS) |
| Apify `harvestapi/linkedin-post-search` | Collecte des posts, sans cookie | ~5 $, en partie couvert par le crédit gratuit |
| Apify `kaitoeasyapi/twitter-x-data-tweet-scraper-pay-per-result-cheapest` | Tweets des comptes suivis, sans compte X | ~0,25 $ / 1 000 tweets, soit 1 à 2 $ |
| SMTP Gmail | Envoi du digest | 0 $ |

Total : **0 à ~12 $ par mois** pour environ 100 posts LinkedIn et 100 tweets analysés par jour.

## Contenu du repo

- `workflow-signaux-toronto.json` : le workflow n8n (LinkedIn + X), à importer tel quel. Après import : choisir la credential Apify (Header Auth `Authorization: Bearer <token>`) dans les deux nœuds Apify, la credential SMTP dans « Envoyer le digest », remplacer `TON_EMAIL`, renseigner `COMPTES_X`, puis activer.
- `docker-compose.yml` : n8n avec fuseau Toronto, port exposé uniquement en local, Cloudflare Tunnel en option.
- `.env.example` : variables d'environnement.

## Limites et conformité

- Le scraping est contraire aux conditions d'utilisation de LinkedIn. Le pipeline reste en lecture seule, à petit volume, et n'utilise **aucun cookie ni compte LinkedIn personnel**. Aucune action (like, commentaire, message) n'est automatisée : l'engagement reste manuel.
- Les données collectées sont des renseignements personnels (LPRPDE au Canada, RGPD en Europe) : garder le minimum, purger régulièrement.
- Les actors Apify sont maintenus par des tiers et peuvent casser quand LinkedIn change son site.

## Pistes

- Scoring plus fin avec Claude Haiku (type de signal, persona, angle d'approche).
- Création automatique de contacts et de tâches dans HubSpot Free pour les scores élevés.
- Ajout de flux RSS de levées de fonds canadiennes.
