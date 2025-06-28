# OxyFoo CI/CD Workflows

Workflows GitHub Actions pour automatiser les déploiements vers le serveur OxyFoo.

## 📚 Sommaire

- [🚀 Workflows disponibles](#-workflows-disponibles)
  - [Deploy Docker Application](#1-deploy-docker-application-deploy-dockeryml)
  - [Deploy Static Website](#2-deploy-static-website-deploy-webstaticyml)
  - [Deploy Temporary Files](#3-deploy-temporary-files-deploy-temporaryyml)
- [🔐 Secrets requis](#-secrets-requis)
- [📋 Exemple complet](#-exemple-complet)
- [🎯 Étapes des workflows](#-étapes-des-workflows)
- [📁 Structure des dossiers serveur](#-structure-des-dossiers-serveur)

## 🚀 Workflows disponibles

### 1. Deploy Docker Application (`deploy-docker.yml`)
Déploie une application Docker avec Docker Compose.

**Utilisation :**
```yaml
jobs:
  deploy:
    uses: ./.github/workflows/deploy-docker.yml
    secrets: inherit
    with:
      author: "mycompany"                              # REQUIS - Nom de l'auteur (minuscules)
      environment: "production"                        # REQUIS - Environnement: development, staging, production
      artifact-name: "my-app"                          # Nom de l'artefact à télécharger (défaut: project-package)
      exclude-folders: ".git,.github,node_modules"     # Dossiers à exclure lors de l'upload (défaut: .git,.github,node_modules)
      docker-compose-file: "docker-compose.prod.yml"   # Fichier Docker Compose à utiliser (défaut: docker-compose.yml)
      docker-requires: "docker.service"                # Service systemd requis (défaut: docker.service)
      run-on-test: "npm run test:e2e"                  # Commandes de test à exécuter
      skip-tests: false                                # Ignorer la phase de test (défaut: false)
      health-check-timeout: 60                         # Timeout pour les vérifications en secondes (défaut: 60)
      skip-verification: false                         # Ignorer la vérification du déploiement (défaut: false)
      restore-on-failure: true                         # Restaurer la sauvegarde en cas d'échec (défaut: true)
      run-on-success: "echo 'Deployment successful'"   # Commandes à exécuter en cas de succès
      run-on-failure: "echo 'Deployment failed'"       # Commandes à exécuter en cas d'échec
```

---

### 2. Deploy Static Website (`deploy-webstatic.yml`)
Déploie un site web statique directement sur le serveur web.

**Utilisation :**
```yaml
jobs:
  deploy:
    uses: ./.github/workflows/deploy-webstatic.yml
    secrets: inherit
    with:
      author: "mycompany"                              # REQUIS - Nom de l'auteur (minuscules)
      web-static-domain: "example.com"                 # REQUIS - Domaine web (ex: example.com)
      web-static-path: "myapp"                         # REQUIS - Chemin sur le serveur (ex: myapp)
      artifact-name: "static-build"                    # Nom de l'artefact à télécharger (défaut: project-package)
      exclude-folders: ".git,.github,node_modules"     # Dossiers à exclure lors de l'upload (défaut: .git,.github,node_modules)
      health-check-url: "https://example.com/myapp"    # URL pour vérifier la disponibilité du site
      health-check-timeout: 60                         # Timeout pour les vérifications en secondes (défaut: 60)
      run-on-test: "npm run test:e2e"                  # Commandes de test à exécuter
      skip-tests: false                                # Ignorer la phase de test (défaut: false)
      skip-verification: false                         # Ignorer la vérification du déploiement (défaut: false)
      run-on-success: "echo 'Website deployed'"        # Commandes à exécuter en cas de succès
      run-on-failure: "echo 'Deployment failed'"       # Commandes à exécuter en cas d'échec
```

---

### 3. Deploy Temporary Files (`deploy-temporary.yml`)
Déploie des fichiers temporaires, supprimés automatiquement après le workflow.

**Utilisation :**
```yaml
jobs:
  deploy:
    uses: ./.github/workflows/deploy-temporary.yml
    secrets: inherit
    with:
      author: "mycompany"                              # REQUIS - Nom de l'auteur (minuscules)
      artifact-name: "temp-files"                      # Nom de l'artefact à télécharger (défaut: project-package)
      exclude-folders: ".git,.github,node_modules"     # Dossiers à exclure lors de l'upload (défaut: .git,.github,node_modules)
      run-on-test: "npm test"                          # Commandes de test à exécuter
      skip-tests: false                                # Ignorer la phase de test (défaut: false)
      skip-verification: false                         # Ignorer la vérification du déploiement (défaut: false)
      run-on-success: "echo 'Tests passed'"            # Commandes à exécuter en cas de succès
      run-on-failure: "echo 'Tests failed'"            # Commandes à exécuter en cas d'échec
```

## 🔑 Permissions requises

Pour utiliser ces workflows, votre workflow appelant doit avoir les permissions suivantes :

```yaml
permissions:
  contents: read      # Lecture du contenu du repository
  actions: write      # Écriture des actions (artefacts)
  packages: write     # Écriture des packages (si nécessaire)
```

## 🔐 Secrets requis

Tous les workflows nécessitent ces secrets :

- `DEPLOY_SERVER_SSH_PRIVATE_KEY` : Clé SSH privée pour se connecter au serveur
- `DEPLOY_SERVER_USER` : Nom d'utilisateur SSH  
- `DEPLOY_SERVER_HOST` : Adresse IP ou nom du serveur
- `DEPLOY_SERVER_PORT` : Port SSH (optionnel, défaut: 22)

### Configuration des secrets

**Si les secrets sont configurés au niveau de l'organisation :**
```yaml
secrets: inherit  # Hérite automatiquement des secrets de l'organisation
```

**Si les secrets sont configurés au niveau du repository :**
```yaml
secrets:
  DEPLOY_SERVER_SSH_PRIVATE_KEY: ${{ secrets.DEPLOY_SERVER_SSH_PRIVATE_KEY }}
  DEPLOY_SERVER_USER: ${{ secrets.DEPLOY_SERVER_USER }}
  DEPLOY_SERVER_HOST: ${{ secrets.DEPLOY_SERVER_HOST }}
  DEPLOY_SERVER_PORT: ${{ secrets.DEPLOY_SERVER_PORT }}  # optionnel
```

## 📋 Exemple complet

<details>
<summary>🔽 Cliquez pour voir l'exemple complet d'un workflow de déploiement</summary>

```yaml
name: Deploy to Production

permissions:
  contents: read
  actions: write
  packages: write

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build project
        run: |
          npm install
          npm run build
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: production-build
          path: dist/

  deploy:
    needs: build
    uses: ./.github/workflows/deploy-docker.yml
    with:
      author: "mycompany"
      environment: "production"
      artifact-name: "production-build"
      run-on-test: "npm run test:e2e"
      health-check-timeout: 120
    secrets:
      DEPLOY_SERVER_SSH_PRIVATE_KEY: ${{ secrets.DEPLOY_SERVER_SSH_PRIVATE_KEY }}
      DEPLOY_SERVER_USER: ${{ secrets.DEPLOY_SERVER_USER }}
      DEPLOY_SERVER_HOST: ${{ secrets.DEPLOY_SERVER_HOST }}
      DEPLOY_SERVER_PORT: ${{ secrets.DEPLOY_SERVER_PORT }}  # optionnel
```

</details>

## 🎯 Étapes des workflows

Chaque workflow suit cette architecture :

1. **🚀 Deploy** : Upload des fichiers vers le serveur
2. **🧪 Test** : Exécution des tests (optionnel)
3. **🚢 Launch** : Lancement des services (Docker uniquement)
4. **✅ Verify** : Vérification du déploiement
5. **🧹 Cleanup** : Nettoyage et finalisation

## 📁 Structure des dossiers serveur

- **Docker** : `/srv/OxyCloud/Projects/{Environment}/{author-project}`
- **WebStatic** : `/srv/OxyCloud/WebServer/domains/{domain}/public_html/{path}`
- **Temporary** : `/srv/OxyCloud/Projects/Temporary/{run_id}`
