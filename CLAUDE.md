# Guetto Platform

Internal Developer Platform de escopo controlado, construída como laboratório de **aprendizado de
Platform Engineering**. Cada repositório da org `guetto-plataform` é uma fronteira de responsabilidade.

## Antes de propor qualquer coisa

Leia, nesta ordem: `docs/README.md` → `docs/adr/README.md` (índice de ADRs) → `docs/roadmap.md`.
As decisões estão nos ADRs; não reabra uma decisão aceita sem um novo ADR que a substitua.

## Princípios do projeto

- Thinnest Viable Platform: componentes entram só com ADR que aponte o problema resolvido (ADR-0001).
- Ambiente o mais **realista** possível, com cada fronteira justificada (dono, permissão, ciclo de vida, consumidor).
- Pesquisar antes de recomendar; citar fontes; separar fato documentado de opinião.
- Documentação em português; termos técnicos consagrados em inglês.

## Repositórios (ADR-0009)

| Repo | Responsabilidade | Fase |
|---|---|---|
| `.github` | Perfil da org + docs transversais (ADRs, arquitetura, aprendizado) | 0 |
| `guetto-infra` | Terraform: bootstrap da conta, rede, EKS, IAM, instalação inicial do Argo CD | 1 |
| `guetto-gitops` | Estado desejado do cluster (add-ons + `apps/<app>/<env>`) | 2 |
| `guetto-charts` | Chart `guetto-app` publicado como OCI com semver | 3 |
| `guetto-ci` | Reusable workflows versionados | 3 |
| `template-*` / `sample-*` | Golden paths (Go e Python) e apps de exemplo | 3 |

Repositórios são criados **somente** na fase em que são necessários.
Biblioteca Terraform externa: `github.com/vitorfprado/terraform-aws-modules`, consumida por tag (ADR-0010).

## Decisões-chave

- AWS `us-east-1`, ambiente **efêmero** (`make up` / `make down`); Budgets e reaper obrigatórios (ADR-0002).
- Conta AWS intercambiável: sem account ID, ARN ou nome de bucket fixos; config por conta em `accounts/<nome>/` (ADR-0003).
- Terraform em camadas: `00-bootstrap` (persistente), `10-network` e `20-cluster` (efêmeros).
  Terraform vai até "cluster + Argo CD"; o resto é GitOps.
- Subnets privadas + NAT Gateway gerenciado (um por padrão).
- EKS em versão com standard support (1.35+); cluster único com namespaces `dev`/`prod` (ADR-0004).
- Argo CD, External Secrets Operator (ADR-0006), GHCR (ADR-0008), GitHub Actions, Go + Python (ADR-0007).

## Convenções de CLAUDE.md

- Cada repositório tem seu próprio `CLAUDE.md` **versionado**, com regras específicas dele (comandos, o que nunca commitar).
- Preferências pessoais e dados do ambiente local (profile AWS, caminhos) ficam em `CLAUDE.local.md`, nunca versionado.
- Repositórios são públicos: nunca incluir account IDs, credenciais ou state.
