GitOps Course Project — Gitea on Kubernetes with FluxCD
Overview
ComponentDetailsApplicationGitea — self-hosted Git serviceDatabasePostgreSQL via CloudNativePG operatorIngressTraefikGitOpsFluxCD
Environments
EnvironmentNamespaceReplicasDB instancesDomainStagingstaging1 (fixed)1gitea.staging.localProductionproduction2–5 (HPA)3gitea.local

Repository Structure
.
├── apps/
│   └── gitea/
│       ├── base/          # shared manifests (HelmRelease, CNPG Cluster, Secrets)
│       ├── staging/       # staging overlay (namespace, ingress host)
│       └── production/    # production overlay (resources, HPA, HA DB)
├── clusters/
│   ├── staging/           # Flux entry point for staging
│   └── production/        # Flux entry point for production
└── infrastructure/
    ├── cnpg/              # CloudNativePG operator (HelmRelease)
    ├── traefik/           # Traefik ingress controller (HelmRelease)
    └── gitea/             # Gitea HelmRepository

Verification
All HelmReleases ready
$ flux get helmreleases -A
NAMESPACE     NAME     REVISION  READY  MESSAGE
flux-system   cnpg     0.23.2    True   Helm install succeeded
flux-system   traefik  33.x.x    True   Helm install succeeded
staging       gitea    10.x.x    True   Helm install succeeded
production    gitea    10.x.x    True   Helm install succeeded
All Kustomizations synced
$ flux get kustomizations -A
NAME              READY  MESSAGE
flux-system       True   Applied revision: main@sha1:...
infrastructure    True   Applied revision: main@sha1:...
apps-staging      True   Applied revision: main@sha1:...
apps-production   True   Applied revision: main@sha1:...
Pods in both namespaces
$ kubectl get pods -A
NAMESPACE    NAME                      READY  STATUS
staging      gitea-xxx                 1/1    Running
staging      gitea-db-1                1/1    Running
production   gitea-xxx                 1/1    Running
production   gitea-db-1                1/1    Running
production   gitea-db-2                1/1    Running
production   gitea-db-3                1/1    Running
Ingress
$ kubectl get ingress -A
NAMESPACE    NAME    CLASS    HOSTS                ADDRESS
staging      gitea   traefik  gitea.staging.local  <node-ip>
production   gitea   traefik  gitea.local          <node-ip>
HPA in production
$ kubectl get hpa -n production
NAME    REFERENCE          MINPODS  MAXPODS  REPLICAS
gitea   Deployment/gitea   2        5        2

Notes

Secrets (postgres-secret.yaml) contain plaintext passwords — acceptable for a course project. In production use Sealed Secrets or External Secrets Operator.
Self-healing: Flux reconciles every 10 minutes. Manually deleted resources are automatically restored.