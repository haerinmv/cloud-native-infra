# Cloud Native Platform — Infrastructure

Repo GitOps géré par ArgoCD.
Toute modification du cluster passe par ce repo.

## Structure
- bootstrap/        → Installation initiale d'ArgoCD
- infrastructure/   → Namespaces, NetworkPolicies, RBAC
- apps/             → Applications déployées par namespace
