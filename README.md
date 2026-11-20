# Signaux LinkedIn + X Toronto

Un pipeline de veille qui repère chaque matin les posts LinkedIn et les tweets X où quelqu'un recrute en RevOps, Sales Ops ou CRM à Toronto (ou en remote au Canada), les fait qualifier par Claude et les envoie dans un e-mail récapitulatif.

## Pourquoi

Les offres d'emploi publiées sur les job boards ne sont qu'une partie du marché. Beaucoup de recrutements commencent par un post : un fondateur qui cherche son premier profil RevOps, un VP Sales qui veut quelqu'un pour reprendre HubSpot. Ces posts disparaissent du fil en quelques heures.

Les outils de « signal-based selling » qui captent ce type de signal coûtent entre 159 $ et 2 500 $ par mois, ou ont fermé (Trigify, octobre 2026). Ce repo montre qu'on peut obtenir l'essentiel pour quelques dollars par mois avec des briques standard.

## Ce que fait le workflow

```
Cron 7 h (Toronto)
  ├→ Apify : posts LinkedIn (5 requêtes booléennes, dernières 24 h)
  └→ Apify : tweets X (3 recherches avancées, les plus récents)
  → Fusion + dédoublonnage (mémoire 30 jours), format commun
  → Claude Haiku : qualification de chaque post (JSON structuré)
  → E-mail : digest HTML des signaux qualifiés, triés par score
```

Claude renvoie pour chaque post un JSON garanti par schéma :

| Champ | Contenu |
|---|---|
| `est_recrutement` | vrai besoin d'embauche ou de prestataire, ou simple bruit |
| `type_signal` | offre directe, mission freelance, besoin implicite, agence |
| `poste`, `entreprise`, `localisation` | tels que mentionnés |
| `persona_auteur` | décideur, manager, recruteur, pair |
| `score` | 0 à 100, adéquation avec le profil ciblé |
| `raison` | une phrase qui justifie le score |
| `angle_approche` | un commentaire ou premier message spécifique au post |

Seuls les vrais recrutements avec un score ≥ 60 arrivent dans le digest.

## Stack et coût

| Brique | Rôle | Coût mensuel |
|---|---|---|
| n8n auto-hébergé (Docker) | Orchestration | 0 $ (serveur gratuit Oracle) à ~6 $ (VPS) |
| Apify `harvestapi/linkedin-post-search` | Posts LinkedIn, sans cookie | ~5 $, en partie couvert par le crédit gratuit |
| Apify `apidojo/tweet-scraper` | Tweets X, sans compte | < 1 $ |
| Claude Haiku 5.5 (API Anthropic) | Qualification des posts | quelques centimes |
| SMTP Gmail | Envoi du digest | 0 $ |

Total : **0 à ~10 $ par mois** pour environ 150 posts analysés par jour.

## Contenu du repo

- `workflow-signaux-toronto.json` : le workflow n8n, à importer tel quel.
- `docker-compose.yml` : n8n avec fuseau Toronto, port exposé uniquement en local, Cloudflare Tunnel en option.
- `.env.example` : variables d'environnement.

## Limites et conformité

- Le scraping est contraire aux conditions d'utilisation de LinkedIn et de X. Le pipeline reste en lecture seule, à petit volume, et n'utilise **aucun cookie ni compte LinkedIn ou X personnel**. Aucune action (like, commentaire, message) n'est automatisée : l'engagement reste manuel.
- Les données collectées sont des renseignements personnels (LPRPDE au Canada, RGPD en Europe) : garder le minimum, purger régulièrement.
- Les actors Apify sont maintenus par des tiers et peuvent casser quand LinkedIn change son site.

## Pistes

- Création automatique de contacts et de tâches dans HubSpot Free pour les scores élevés.
- Ajout de flux RSS de levées de fonds canadiennes.
- Revue hebdomadaire des meilleurs signaux par une routine Claude.
