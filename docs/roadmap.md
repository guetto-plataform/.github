# Roadmap

> Cada fase termina com algo **funcionando** e testável. Critérios detalhados serão expandidos ao iniciar cada fase.

| Fase | Objetivo | Critério de conclusão | Status |
|---|---|---|---|
| 0 | Definição: problema, métricas, ADRs iniciais, contexto C4 | Explicar em 2 min o que a plataforma faz e não faz | Em andamento |
| 1 | AWS efêmera: bootstrap, rede, EKS por Terraform; `make up`/`make down` | Subir e derrubar 3× seguidas sem órfãos; custo medido | — |
| 2 | GitOps: Argo CD, App of Apps (plataforma), ApplicationSet (apps), ESO, LB Controller | Recriar o ambiente e tudo convergir sozinho; self-heal e prune demonstrados | — |
| 3 | Golden path: chart `guetto-app`, templates Go/Python, reusable workflows, GHCR | Serviço novo em `dev` ≤ 15 min; rollback ≤ 5 min | — |
| 4 | Observabilidade operacional: OTel, métricas, logs, 1 SLO + burn-rate, runbooks | Game day: 3 falhas injetadas diagnosticadas via runbook | — |
| 5 | Guardrails: políticas de admissão, RBAC por AppProject, NetworkPolicy, scan, threat model | Manifest inválido rejeitado com mensagem útil | — |
| 6 | Release seguro: Argo Rollouts **ou** Kargo (escolha via ADR) | Versão ruim revertida sem intervenção | — |
| 7+ | Sob demanda, com ADR: multi-cluster, CLI/operator em Go, Backstage, IA | Definido no ADR | — |

## Fora do escopo até justificativa

Cluster de gerenciamento, API própria, Crossplane, portal, IA.
