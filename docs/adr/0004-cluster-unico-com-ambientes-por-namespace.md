# ADR-0004: Cluster único com ambientes `dev` e `prod` por namespace

- **Status:** Proposto
- **Data:** 2026-10-13
- **Fase do roadmap:** 2

## Contexto

A hipótese inicial incluía um cluster de gerenciamento controlando clusters de workload (hub-spoke).
Não há hoje requisito de isolamento, região ou blast radius que justifique múltiplos clusters, e cada cluster EKS
adiciona ~US$ 0,10/h de control plane.

## Alternativas consideradas

| Alternativa | Isolamento | Custo | Complexidade |
|---|---|---|---|
| **1 cluster, namespaces por ambiente** | Fraco (RBAC, quotas, NetworkPolicy) | 1× | Baixa |
| 1 cluster por ambiente, Argo CD em cada | Forte | 2× | Média |
| Hub-spoke (Argo CD central) | Forte; hub é ponto único de falha | 3× | Alta (bootstrapping do hub, credenciais cross-cluster) |

## Decisão (proposta)

Um cluster EKS, namespaces `dev` e `prod`. App of Apps para componentes da plataforma (uso admin) e
ApplicationSet (git directory generator) para as aplicações.

## Quando revisar

Requisito explícito como "testar upgrade de Kubernetes sem afetar prod" ou "prod não compartilha control plane".
Nesse momento, ADR comparando hub-spoke, Argo CD por cluster e argocd-agent.

## Referências

- [Argo CD — cluster bootstrapping (App of Apps é "admin-only")](https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/)
- [Argo CD — segurança de ApplicationSets](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/Security/)
- [Arquiteturas de Argo CD (Octopus)](https://octopus.com/blog/a-comprehensive-overview-of-argo-cd-architectures)
- [argocd-agent](https://argocd-agent.readthedocs.io/latest/features/)
