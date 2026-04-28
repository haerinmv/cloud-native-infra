# Cloud Native Platform : Infrastructure

Repo GitOps géré par ArgoCD. Toute modification du cluster 
passe par ce repo soit aucune manipulation directe en production.

Le but est de simuler une situation professionelle et surtout de comprendre les outils d'un ingenieur cloud/devops.


## Stack technique

| Outil | Rôle |
|-------|------|
| k3s   | Distribution Kubernetes légère (optimisé pour ARM) |
| Calico| CNI + NetworkPolicies (microsegmentation Zero-Trust) |
| ArgoCD| GitOps, déploiement automatisé depuis Git (Pull Model) |
| Vault | (HashiCorp) Gestion centralisée et dynamique des mots de passe |
| Sealed Secrets | Chiffrement asymétrique des secrets dans Git |
| MinIO | Stockage objet S3-compatible |
| Prometheus + Grafana | Monitoring et alerting |
| Loki + Promtail | Centralisation et aggrégation des logs |
| Keycloak | Authentification SSO via protocoles OIDC/OAuth2 |
| Kyverno | Policy-as-Code (Contrôleur d'admission K8s) |
| Falco | Intrusion Detection System (Analyse noyau eBPF) |

## Architecture sécurité

- **Deny-all par défaut** sur chaque namespace
- Ouverture explicite des flux nécessaires uniquement (Ingress/Egress sur ports spécifiques)
- Transistion vers le **Dynamic Secrets management** avec Vault pour éviter l'exposition des secrets Kubernetes dans etcd
- **Container Hardening** rigoureux validé par Trivy (Drop Capabilities, RunAsNonRoot, ReadOnlyRootFilesystem)
- **Policy-as-code** avec Kyverno pour imposer des standards stricts (rejet systématique des images utilisant la balise "latest" en production)
- Détection d'anomalies comportementales à l'échelle du noyau Linux grâce à Falco

## Structure GitOps (App of Apps Pattern)

```bash
├── apps
│   ├── ci-cd
│   ├── dev
│   ├── logging
│   │   ├── loki.yaml
│   │   └── promtail.yaml
│   ├── monitoring
│   │   └── kube-prometheus-stack.yaml
│   ├── networking
│   ├── prod
│   │   ├── keycloak-sealed-secret.yaml
│   │   ├── keycloak.yaml
│   │   ├── minio-sealed-secret.yaml
│   │   └── minio.yaml
│   └── security
│       ├── falco.yaml
│       ├── kyverno-policies.yaml
│       ├── kyverno.yaml
│       ├── sealed-secrets.yaml
│       └── vault.yaml
├── bootstrap
│   ├── infrastructure-app.yaml
│   ├── logging-app.yaml
│   ├── monitoring-app.yaml
│   └── prod-app.yaml
├── infrastructure
│   ├── namespaces
│   │   └── namespaces.yaml
│   ├── network-policies
│   │   ├── ci-cd-allow.yaml
│   │   ├── deny-all.yaml
│   │   ├── monitoring-allow.yaml
│   │   └── prod-allow.yaml
│   ├── rbac
│   └── storage
└── README.md
```

## Workflow GitOps

Ajout ou modif d'un fichier YAML sur mon PC , git push GitHub sur une branche de feature , Validation CI par GitHub Action , ArgoCD détecte la différence donc soit du Prune soit du Heal et enfin le déploiement automatique sur le cluster

## Pipeline Intégration Continue (CI)

Pour protéger le cluster GitOps contre les erreurs humaines, une Pipeline via **GitHub Actions** s'exécute à chaque commit. L'objectif est d'empêcher tout code défaillant ou non sécurisé d'atteindre le cluster.

- **Yamllint** : Analyse chaque fichier pour y imposer une norme d'indentation stricte. Cela garantit un référentiel propre et assure l'homogénéité du code en équipe.
- **Kubeconform** : Scanne les définitions de l'infrastructure et les compare au dictionnaire officiel de Kubernetes. Il garantit qu'aucune API obsolète ou structure YAML invalide ne franchisse la porte de production.
- **Trivy** : Réalise des audits de misconfigurations Kubernetes complets (analyse des Kubernetes Security Vulnerabilities : KSV). Il vérifie notamment que chaque pod suit les principes d'isolation (NonRoot, systèmes de fichiers en lecture seule, abandon des privilèges kernel).


## Projet pour la suite


Pour la suite je pensais a coder en go ou python soit un scanner de faille interne a mon cluster ou un binaire. 

La suite sera directement mis a jour ici.
