# Cloud Native Platform : Infrastructure

Repo GitOps géré par ArgoCD. Toute modification du cluster 
passe par ce repo soit aucune manipulation directe en production.

Le but est de simuler une situation professionelle et surtout de comprendre les outils d'un ingenieur cloud/devops.

> Documentation complète disponible sur 
> [Cloud-Native-Platform](https://github.com/haerinmv/Cloud-Native-Platform)

## Stack technique

| Outil | Rôle |
|-------|------|
| k3s   | Distribution Kubernetes légère |
| Calico| CNI + NetworkPolicies (microsegmentation) |
| ArgoCD| GitOps , déploiement automatisé depuis Git |
| Sealed Secrets | Chiffrement des secrets dans Git |
| MinIO | Stockage objet S3-compatible |
| Prometheus + Grafana | Monitoring et métriques |
| Loki + Promtail | Centralisation des logs |
| Keycloak | Authentification SSO  |

## Architecture sécurité

- **Deny-all par défaut** sur chaque namespace
- Ouverture explicite des flux nécessaires uniquement
- Secrets chiffrés via Sealed Secrets donc repo public sans risque
- RBAC configuré par serviceaccount

## Structure

```bash
├── apps
│   ├── ci-cd
│   ├── dev
│   ├── logging
│   │   ├── loki.yaml
│   │   └── promtail.yaml
│   ├── monitoring
│   │   └── kube-prometheus-stack.yaml
│   ├── networking
│   ├── prod
│   │   ├── keycloak-sealed-secret.yaml
│   │   ├── keycloak.yaml
│   │   ├── minio-sealed-secret.yaml
│   │   └── minio.yaml
│   └── security
│       └── sealed-secrets.yaml
├── bootstrap
│   ├── infrastructure-app.yaml
│   ├── logging-app.yaml
│   ├── monitoring-app.yaml
│   └── prod-app.yaml
├── infrastructure
│   ├── namespaces
│   │   └── namespaces.yaml
│   ├── network-policies
│   │   ├── ci-cd-allow.yaml
│   │   ├── deny-all.yaml
│   │   ├── monitoring-allow.yaml
│   │   └── prod-allow.yaml
│   ├── rbac
│   └── storage
└── README.md
```

## Workflow GitOps


Ajout ou modif d'un fichier YAML sur PC
        →
git push GitHub
        →
ArgoCD détecte la différence
        →
Déploiement automatique sur le cluster

## Pipeline 

Pour protéger le cluster GitOps contre les erreurs humaines j'ai mis en place une Pipeline via **GitHub Actions** qui s'exécute à chaque commit. L'objectif est d'empêcher tout code défaillant d'atteindre le serveur , on a donc :
- **Yamllint** : Analyse chaque fichier pour y imposer une norme d'indentation stricte et traquer les mauvaises pratiques syntaxiques. Cela garantit un référentiel propre et lisible en équipe.
- **Kubeconform** : Scanne les définitions de l'infrastructure et les compare au dictionnaire officiel de Kubernetes. Il garantit qu'aucune erreur de frappe ou API obsolète ne viennent silencieusement tout casser.
-**Trivy** : Réalise des test de misconfiguration pour trouver des failles et compare avec les CVE connues.


## Disclamer

Les dossiers vides sont soit des dossiers auquel j'ai configurer avant la 
mise en place de argocd tel que ci-cd et le reste j'ai pas encore finit de deployer en prod par exemple.
