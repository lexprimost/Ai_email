# Guide de déploiement avec Dokploy

Ce guide vous explique comment déployer votre application 0.email avec Dokploy étape par étape.

## Prérequis

- Un serveur avec Docker et Dokploy installé
- Un accès SSH à votre serveur
- Un domaine configuré (optionnel mais recommandé)
- Les clés API nécessaires (voir section Variables d'environnement)

## Étape 1 : Préparation du projet

### 1.1 Vérifier la structure du projet

Assurez-vous que votre projet contient :
- `docker/app/Dockerfile` - pour l'application principale
- `docker/db/Dockerfile` - pour les migrations
- `docker-compose.prod.yaml` - configuration de production

### 1.2 Préparer le repository Git

Votre code doit être dans un repository Git (GitHub, GitLab, etc.) accessible depuis votre serveur Dokploy.

## Étape 2 : Configuration dans Dokploy

### 2.1 Créer un nouveau projet

1. Connectez-vous à votre interface Dokploy
2. Cliquez sur **"Nouveau projet"** ou **"New Project"**
3. Choisissez **"Docker Compose"** comme type de déploiement

### 2.2 Configurer le repository Git

1. **Nom du projet** : `zero-email` (ou le nom de votre choix)
2. **Repository URL** : L'URL de votre repository Git
   - Exemple : `https://github.com/votre-username/Ai_email.git`
3. **Branch** : `main` ou `master` (selon votre branche principale)
4. **Build Path** : `/` (racine du projet)

### 2.3 Configuration Docker Compose

Dans la section **"Docker Compose File"**, spécifiez :
- **Compose File Path** : `docker-compose.prod.yaml`

## Étape 3 : Configuration des services

### 3.1 Service principal (zero)

Le service `zero` est votre application principale. Dokploy le détectera automatiquement depuis votre `docker-compose.prod.yaml`.

### 3.2 Service de base de données (db)

Le service `db` utilise PostgreSQL. Dokploy créera automatiquement le volume pour la persistance des données.

### 3.3 Service de cache (valkey)

Le service `valkey` (Redis) sera également déployé automatiquement.

### 3.4 Service proxy (upstash-proxy)

Le service `upstash-proxy` sera configuré automatiquement.

## Étape 4 : Variables d'environnement

Configurez les variables d'environnement suivantes dans Dokploy :

### Variables obligatoires

```env
# Base de données
POSTGRES_USER=postgres
POSTGRES_PASSWORD=votre_mot_de_passe_securise
POSTGRES_DB=zerodotemail

# URLs de l'application
NEXT_PUBLIC_BACKEND_URL=https://votre-backend-url.com
NEXT_PUBLIC_APP_URL=https://votre-domaine.com

# Redis/Valkey
REDIS_URL=redis://valkey:6379
REDIS_TOKEN=votre_token_securise

# API Keys
RESEND_API_KEY=votre_cle_resend
GROQ_API_KEY=votre_cle_groq
PERPLEXITY_API_KEY=votre_cle_perplexity
OPENAI_API_KEY=votre_cle_openai
OPENAI_MODEL=gpt-4
OPENAI_MINI_MODEL=gpt-3.5-turbo

# ElevenLabs
NEXT_PUBLIC_ELEVENLABS_AGENT_ID=votre_agent_id

# Image Proxy
NEXT_PUBLIC_IMAGE_PROXY=https://votre-image-proxy.com

# PostHog (analytics)
NEXT_PUBLIC_POSTHOG_KEY=votre_cle_posthog
NEXT_PUBLIC_POSTHOG_HOST=https://app.posthog.com

# Image API
NEXT_PUBLIC_IMAGE_API_URL=https://votre-image-api.com

# AI System Prompt (optionnel)
AI_SYSTEM_PROMPT=Votre prompt système pour l'IA
```

### Comment ajouter les variables dans Dokploy

1. Allez dans la section **"Environment Variables"** de votre projet
2. Cliquez sur **"Add Variable"**
3. Ajoutez chaque variable une par une
4. Cochez **"Secret"** pour les clés API sensibles

## Étape 5 : Configuration du port et du domaine

### 5.1 Port de l'application

L'application écoute sur le port **3000** par défaut. Dans Dokploy :

1. Allez dans **"Ports"** ou **"Network"**
2. Configurez le port **3000** pour le service `zero`
3. Dokploy peut mapper automatiquement ce port ou vous pouvez spécifier un port personnalisé

### 5.2 Configuration du domaine (optionnel)

1. Allez dans **"Domains"** ou **"Custom Domain"**
2. Ajoutez votre domaine : `votre-domaine.com`
3. Configurez les enregistrements DNS :
   - **A Record** : Pointez vers l'IP de votre serveur Dokploy
   - **CNAME** : Ou utilisez un CNAME si Dokploy le supporte

## Étape 6 : Volumes persistants

Dokploy devrait automatiquement créer les volumes suivants depuis votre `docker-compose.prod.yaml` :

- `postgres-data` : Pour la base de données PostgreSQL
- `valkey-data` : Pour le cache Valkey

Vérifiez dans la section **"Volumes"** que ces volumes sont bien créés.

## Étape 7 : Déploiement

### 7.1 Premier déploiement

1. Cliquez sur **"Deploy"** ou **"Déployer"**
2. Dokploy va :
   - Cloner votre repository
   - Construire les images Docker
   - Démarrer tous les services dans l'ordre correct
   - Exécuter les migrations de base de données

### 7.2 Ordre de démarrage

Les services démarreront dans cet ordre (grâce aux `depends_on` dans docker-compose) :

1. **db** (PostgreSQL) - attend que la base soit prête
2. **valkey** (Redis) - attend que le cache soit prêt
3. **upstash-proxy** - attend que le proxy soit prêt
4. **migrations** - s'exécute une fois que la base est prête
5. **zero** - démarre après toutes les dépendances

### 7.3 Vérification du déploiement

1. Attendez que tous les services soient **"Healthy"** ou **"Running"**
2. Vérifiez les logs dans la section **"Logs"**
3. Testez l'application en accédant à votre URL

## Étape 8 : Configuration post-déploiement

### 8.1 Vérifier les migrations

Les migrations s'exécutent automatiquement au démarrage. Vérifiez les logs du service `migrations` pour confirmer qu'elles ont réussi.

### 8.2 Vérifier la santé de l'application

L'application a un healthcheck configuré. Vérifiez dans Dokploy que le service `zero` est marqué comme **"Healthy"**.

### 8.3 Tester l'application

1. Accédez à votre URL : `https://votre-domaine.com` ou `http://votre-ip:port`
2. Vérifiez que l'application se charge correctement
3. Testez les fonctionnalités principales

## Étape 9 : Configuration SSL/HTTPS (recommandé)

### 9.1 Avec Dokploy

Si Dokploy supporte Let's Encrypt :

1. Allez dans **"SSL"** ou **"Certificates"**
2. Activez **"Let's Encrypt"**
3. Entrez votre domaine
4. Dokploy générera automatiquement le certificat SSL

### 9.2 Avec un reverse proxy externe

Si vous utilisez un reverse proxy (Nginx, Traefik, etc.) :

1. Configurez le reverse proxy pour pointer vers le port 3000
2. Configurez SSL dans le reverse proxy
3. Mettez à jour `NEXT_PUBLIC_APP_URL` avec l'URL HTTPS

## Étape 10 : Monitoring et logs

### 10.1 Logs

Dans Dokploy, vous pouvez voir les logs de chaque service :
- **zero** : Logs de l'application principale
- **db** : Logs de PostgreSQL
- **valkey** : Logs du cache
- **migrations** : Logs des migrations (une seule fois)

### 10.2 Monitoring

Configurez des alertes dans Dokploy pour :
- Services qui redémarrent fréquemment
- Utilisation élevée de la mémoire/CPU
- Erreurs dans les logs

## Étape 11 : Mises à jour et redéploiements

### 11.1 Déploiement automatique

Si configuré, Dokploy peut redéployer automatiquement lors des push sur votre branche principale.

### 11.2 Déploiement manuel

1. Poussez vos changements sur Git
2. Dans Dokploy, cliquez sur **"Redeploy"** ou **"Rebuild"**
3. Attendez la fin du déploiement

### 11.3 Rollback

Si quelque chose ne va pas :
1. Allez dans **"Deployments"** ou **"History"**
2. Sélectionnez une version précédente
3. Cliquez sur **"Rollback"**

## Dépannage

### Problème : Les migrations échouent

**Solution** :
- Vérifiez que `DATABASE_URL` est correctement configuré
- Vérifiez les logs du service `migrations`
- Assurez-vous que la base de données est accessible

### Problème : L'application ne démarre pas

**Solution** :
- Vérifiez les logs du service `zero`
- Vérifiez que toutes les variables d'environnement sont définies
- Vérifiez que les services dépendants (db, valkey) sont démarrés

### Problème : Erreur de connexion à la base de données

**Solution** :
- Vérifiez que `POSTGRES_USER`, `POSTGRES_PASSWORD`, et `POSTGRES_DB` sont corrects
- Vérifiez que le service `db` est en cours d'exécution
- Vérifiez que `DATABASE_URL` utilise le bon format : `postgresql://user:password@db:5432/database`

### Problème : Erreur de build Docker

**Solution** :
- Vérifiez que tous les fichiers nécessaires sont dans le repository
- Vérifiez les logs de build pour des erreurs spécifiques
- Assurez-vous que `pnpm-lock.yaml` est à jour

## Commandes utiles

### Vérifier l'état des services

Dans Dokploy, utilisez l'interface pour voir l'état de tous les services.

### Accéder aux logs

Utilisez la section **"Logs"** dans Dokploy pour voir les logs en temps réel.

### Redémarrer un service

Dans Dokploy, vous pouvez redémarrer un service individuel depuis l'interface.

## Notes importantes

1. **Sécurité** : Ne commitez jamais vos variables d'environnement dans Git. Utilisez toujours la configuration de Dokploy.

2. **Backups** : Configurez des backups réguliers pour le volume `postgres-data`.

3. **Performance** : Surveillez l'utilisation des ressources (CPU, RAM) et ajustez si nécessaire.

4. **Mises à jour** : Gardez vos images Docker à jour pour la sécurité.

5. **Monitoring** : Configurez des alertes pour être notifié en cas de problème.

## Support

Pour plus d'aide :
- Documentation Dokploy : [https://dokploy.com/docs](https://dokploy.com/docs)
- Issues GitHub de votre projet
- Logs de déploiement dans Dokploy

---

**Dernière mise à jour** : Février 2025

