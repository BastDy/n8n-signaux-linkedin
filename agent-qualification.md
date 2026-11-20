# Agent de qualification des signaux

L'agent Claude utilisé par le nœud **« Claude : qualifier le post »** de `workflow-signaux-toronto.json`. Il lit chaque post LinkedIn ou tweet X et renvoie une qualification en JSON.

> Ce fichier est une copie de référence. La version qui tourne est celle du workflow : en cas de modification, mettre à jour les deux.

## Réglages

| Paramètre | Valeur |
|---|---|
| Appel | `POST https://api.anthropic.com/v1/messages` (nœud HTTP Request, identifiant `anthropicApi`) |
| En-tête | `anthropic-version: 2026-10-21` |
| Modèle | `claude-haiku-5-5` |
| `max_tokens` | 2048 |
| `effort` | `low` |
| Sortie | JSON garanti par schéma (`output_config.format`) |
| Lots | 5 posts par lot, timeout 120 s |

## Prompt système

```text
Tu qualifies des posts LinkedIn et des tweets X pour un professionnel RevOps basé à Toronto.

Son profil : Revenue Operations / Sales Ops, expert HubSpot (et CRM en général), automatisation GTM (n8n, Clay, enrichissement). Il cherche un poste RevOps, Sales Ops, Marketing Ops, CRM ou GTM à Toronto/GTA ou en remote au Canada, et accepte aussi des missions freelance.

Pour chaque post, détermine :
- est_recrutement : true seulement si le post annonce un vrai besoin d'embauche ou de prestataire (pas un conseil de carrière, une annonce d'avoir été embauché, un webinaire, une liste d'astuces).
- type_signal : offre_emploi_directe, recherche_freelance, besoin_implicite, recruteur_agence ou autre.
- poste, entreprise, localisation : tels que mentionnés, sinon chaîne vide.
- persona_auteur : decideur (fondateur, C-level, VP, Head of, Director), manager, recruteur, pair ou autre.
- score de 0 à 100 : adéquation avec le profil. 80-100 = poste ou mission RevOps/HubSpot à Toronto ou remote Canada publié par un décideur ; 60-79 = bonne adéquation avec un défaut ; 30-59 = lien faible ; 0-29 = hors sujet.
- raison : une phrase en français qui justifie le score.
- angle_approche : une phrase en français qui propose un commentaire ou un premier message concret et spécifique au post.
```

## Message utilisateur (un par post)

Construit à partir de la sortie du nœud « Dédoublonner et préparer » :

```text
Source : {{ $json.source }}
Auteur : {{ $json.author }}
Titre : {{ $json.headline }}
Publié : {{ $json.postedAgo }}

Post :
{{ $json.text }}
```

## Schéma de sortie

```json
{
  "type": "object",
  "additionalProperties": false,
  "required": ["est_recrutement", "type_signal", "poste", "entreprise", "localisation", "persona_auteur", "score", "raison", "angle_approche"],
  "properties": {
    "est_recrutement": { "type": "boolean" },
    "type_signal": { "type": "string", "enum": ["offre_emploi_directe", "recherche_freelance", "besoin_implicite", "recruteur_agence", "autre"] },
    "poste": { "type": "string" },
    "entreprise": { "type": "string" },
    "localisation": { "type": "string" },
    "persona_auteur": { "type": "string", "enum": ["decideur", "manager", "recruteur", "pair", "autre"] },
    "score": { "type": "integer" },
    "raison": { "type": "string" },
    "angle_approche": { "type": "string" }
  }
}
```

Le nœud « Construire le digest » ne garde ensuite que les posts avec `est_recrutement = true` et `score ≥ 60`.
