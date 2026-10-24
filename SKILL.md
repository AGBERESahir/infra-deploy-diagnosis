---
name: infra-deploy-diagnosis
description: Procédure de diagnostic de pannes d'infrastructure et d'erreurs de déploiement - lire les logs dans le bon ordre, vérifier réseau, DNS, TLS, permissions, variables d'environnement, ressources, puis remonter la chaîne. À utiliser quand un déploiement échoue, un site ou une API est inaccessible, erreurs 502/503/504, CORS, "connection refused", timeout, conteneur Docker qui redémarre, CI/CD en échec, problème Nginx, Vercel, Netlify, Supabase, AWS ou serveur Linux, même si l'utilisateur décrit seulement un symptôme vague. Use for outages, deployment failures, network and server troubleshooting.
---

# Diagnostic d'infrastructure et de déploiement

Les pannes de déploiement ont des causes récurrentes. Suivre la chaîne de requête de bout en bout évite de chercher au mauvais endroit. Principe : **localiser la couche en défaut avant de chercher pourquoi**.

## 1. Cadrer
Poser les faits : quoi est en panne (tout ou une partie), depuis quand, **qu'est-ce qui a changé** (déploiement, configuration, certificat expiré, quota, facture, mise à jour), qui est touché (tout le monde ou une région/un réseau), message d'erreur exact. Demander les logs et la plateforme concernée si absents.

## 2. Suivre la chaîne, de l'extérieur vers l'intérieur

| Couche | Questions à vérifier | Outils typiques |
|---|---|---|
| DNS | Le nom résout-il vers la bonne IP ? Propagation terminée ? | `dig`, `nslookup` |
| Réseau | Le port est-il joignable ? Pare-feu, groupe de sécurité ? | `curl -v`, `nc -zv`, `traceroute` |
| TLS | Certificat valide, non expiré, chaîne complète, bon nom de domaine ? | `openssl s_client`, `curl -vI` |
| Proxy / load balancer | Bon upstream ? Health check au vert ? Timeouts ? | logs Nginx/Traefik/LB |
| Application | Le processus tourne-t-il ? Écoute-t-il sur la bonne interface (0.0.0.0) et le bon port ? | `ps`, `ss -tlnp`, logs applicatifs |
| Configuration | Variables d'environnement présentes **dans l'environnement du déploiement** ? Bon fichier de config ? | `printenv`, console de la plateforme |
| Ressources | Disque plein, mémoire (OOM kill), CPU, descripteurs de fichiers, quotas ? | `df -h`, `free -m`, `dmesg`, `docker stats` |
| Dépendances | Base de données joignable ? Identifiants valides ? Services tiers en panne ? | test de connexion direct, page de statut |
| Permissions | Droits sur fichiers, rôles IAM, politiques d'accès (RLS, CORS), utilisateur du processus | `ls -l`, journaux d'accès |

## 3. Lire les logs dans le bon ordre
1. Chercher la **première** erreur chronologique, pas la dernière : les suivantes sont souvent des conséquences.
2. Regarder les logs juste avant le premier symptôme, pas seulement au moment du crash.
3. Corréler plusieurs sources (proxy, application, base de données) par horodatage.
4. Pour un build en échec : lire depuis le début l'étape qui échoue, vérifier versions de runtime, cache, lockfile.

## 4. Causes fréquentes par symptôme
- **502/504** : application arrêtée ou trop lente, mauvais port d'écoute, timeout du proxy
- **Connection refused** : service non démarré, mauvais port, écoute sur 127.0.0.1 seulement, pare-feu
- **Fonctionne en local, pas en production** : variable d'environnement manquante, version de runtime différente, chemin absolu, différences de casse des noms de fichiers (Linux sensible à la casse), dépendance de développement absente
- **CORS** : origine non autorisée côté serveur, en-têtes de pré-requête (OPTIONS) mal gérés, mélange HTTP/HTTPS
- **Conteneur qui redémarre en boucle** : commande de démarrage en erreur, OOM kill, health check trop strict, volume manquant
- **Erreurs intermittentes** : épuisement de connexions à la base, limite de débit, instance en mauvaise santé derrière le load balancer
- **Soudainement en panne sans déploiement** : certificat expiré, domaine non renouvelé, clé/token expiré, quota atteint, disque rempli par les logs

## 5. Conduite
- Faire une vérification à la fois et noter le résultat ; ne pas redéployer "au hasard".
- **Prudence sur les actions destructrices** (redémarrage de production, suppression de volumes, modification de pare-feu) : les annoncer et demander confirmation.
- Ne jamais demander de coller des secrets en clair ; demander de masquer les valeurs sensibles dans les logs partagés.
- Si l'outil bash et le réseau le permettent, tester directement (`curl`, `dig`) plutôt que supposer.

## Format de réponse
```
## Hypothèse sur la couche en défaut
## Vérifications à faire (dans l'ordre, avec les commandes)
## Interprétation des résultats possibles
## Correctif et prévention (supervision, alertes, expiration de certificat, etc.)
```
