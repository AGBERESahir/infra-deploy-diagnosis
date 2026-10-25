# infra-deploy-diagnosis

Donne à Claude une **procédure de diagnostic** pour les pannes d'infrastructure et les déploiements en échec : localiser d'abord la **couche en défaut**, puis chercher pourquoi. Les pannes de déploiement ont des causes récurrentes, et suivre la chaîne de bout en bout évite de chercher au mauvais endroit.

Skill Claude : un dossier d'instructions que Claude charge automatiquement quand ta demande correspond. Tu n'as rien à invoquer : décris ton problème normalement.

## Quand il se déclenche

- un déploiement échoue ou un site ou une API est inaccessible ;
- tu vois des erreurs 502, 503, 504, CORS, « connection refused » ou un timeout ;
- un conteneur Docker redémarre en boucle, un pipeline CI/CD échoue, Nginx, Vercel, Netlify, Supabase, AWS ou un serveur Linux se comporte mal ;
- tu ne décris qu'un symptôme vague (« ça ne marche plus en production »).

Exemples de messages :

- « Mon site renvoie une 502 depuis ce matin, je n'ai rien changé. »
- « Le déploiement Vercel échoue alors que ça marche en local. »
- « Mon conteneur redémarre en boucle, voici les logs. »

## Ce que le skill fait faire à Claude

1. **Cadrer** : quoi est en panne (tout ou une partie), depuis quand, **ce qui a changé**, qui est touché, message d'erreur exact.
2. **Suivre la chaîne de l'extérieur vers l'intérieur**, avec l'outil adapté à chaque couche : DNS (`dig`), réseau (`curl -v`, `nc -zv`), TLS (`openssl s_client`), proxy ou load balancer, application (`ss -tlnp`), configuration (variables d'environnement du déploiement), ressources (`df -h`, `free -m`, `dmesg`), dépendances, permissions.
3. **Lire les logs dans le bon ordre** : la *première* erreur chronologique, pas la dernière ; les logs juste avant le symptôme ; plusieurs sources corrélées par horodatage.
4. **Partir des causes fréquentes par symptôme** : 502/504, connexion refusée (écoute sur 127.0.0.1 seulement), « marche en local mais pas en production » (variable manquante, casse des noms de fichiers), CORS, conteneur en boucle (OOM), erreurs intermittentes, panne soudaine sans déploiement (certificat expiré, quota, disque plein).
5. **Conduire proprement** : une vérification à la fois, pas de redéploiement au hasard, actions destructrices annoncées avec demande de confirmation, jamais de secrets collés en clair.

## Format de la réponse

```
## Hypothèse sur la couche en défaut
## Vérifications à faire (dans l'ordre, avec les commandes)
## Interprétation des résultats possibles
## Correctif et prévention (supervision, alertes, expiration de certificat)
```

## Installation

**Claude Code**

```bash
git clone https://github.com/AGBERESahir/infra-deploy-diagnosis.git ~/.claude/skills/infra-deploy-diagnosis
```

Le dossier doit s'appeler `infra-deploy-diagnosis` et contenir `SKILL.md` à sa racine. Relance Claude Code si une session était ouverte.

**Claude.ai**

Zippe le dossier `infra-deploy-diagnosis` (avec `SKILL.md` dedans), puis ajoute l'archive dans **Paramètres > Skills**.

## Vérifier qu'il fonctionne

Envoie l'un des messages d'exemple ci-dessus. La réponse doit suivre la structure décrite plus haut. Si Claude répond de façon libre, vérifie que le skill est activé dans tes paramètres et que `SKILL.md` n'est pas dans un sous-dossier.

## Limites

- Claude diagnostique à partir de ce que tu décris et des logs que tu colles. Il ne voit pas ton serveur : les commandes sont à lancer chez toi, puis à lui rapporter.
- **Masque les secrets** (mots de passe, tokens, clés) avant de coller des logs ou des fichiers de configuration.
- Le skill n'applique aucun changement de lui-même : toute action sur la production (redémarrage, suppression de volume, pare-feu) reste ta décision.
- Une panne sur un service tiers (hébergeur, fournisseur) ne se corrige pas depuis chez toi : vérifie sa page de statut.

## Voir aussi

- [systematic-debugging](https://github.com/AGBERESahir/systematic-debugging) : quand la panne vient du code, pas de l'infrastructure.
- [performance-optimization](https://github.com/AGBERESahir/performance-optimization) : pour les lenteurs et la montée en charge.
- [code-security-review](https://github.com/AGBERESahir/code-security-review) : pour vérifier la configuration avant mise en production.

## Fichiers

| Fichier | Rôle |
|---|---|
| `SKILL.md` | Les instructions : nom, description de déclenchement, démarche et format de sortie. |
| `README.md` | Cette page, pour les humains (Claude ne la lit pas). |

## Licence

Aucune licence n'est encore indiquée dans ce dépôt ; ajoute un fichier `LICENSE` avant de le réutiliser ou de le partager.
