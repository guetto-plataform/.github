# ADR-0009: Multirepo separado por fronteira de responsabilidade

- **Status:** Aceito
- **Data:** 2026-10-13
- **Fase do roadmap:** 0

## Contexto

A proposta inicial era um monorepo `guetto-platform` (infra + gitops + charts + workflows), otimizado para manutenção
por uma pessoa. O objetivo do projeto, porém, é **aprendizado próximo da realidade**, e os problemas mais
característicos de Platform Engineering aparecem justamente nas fronteiras entre repositórios:
versionamento de artefatos compartilhados, credenciais cross-repo, passagem de dados entre IaC e GitOps,
permissões distintas por tipo de mudança.

## Critério de separação

Não "porque empresas separam", e sim: **um repositório por fronteira real de**
(1) dono, (2) permissão/blast radius, (3) ciclo de vida/cadência de release, (4) consumidor.

## Alternativas consideradas

| Alternativa | Prós | Contras |
|---|---|---|
| Monorepo da plataforma | Mudanças atômicas, um PR, pouca cerimônia | Esconde os problemas de integração que se quer aprender; mistura permissões (AWS admin x config de app) |
| **Multirepo por fronteira** | Fronteiras explícitas; exercita versionamento, contratos e credenciais entre repos | Mais overhead; mudanças coordenadas em vários PRs |
| Um repo por componente (ex.: um por add-on) | — | Fragmentação artificial; nenhuma fronteira real justifica |

## Decisão

| Repositório | Dono (simulado) | Responsabilidade | Consome | Criado na fase |
|---|---|---|---|---|
| `.github` | Plataforma | Perfil da org e documentação transversal (ADRs, arquitetura, aprendizado) | — | 0 |
| `guetto-infra` | Infra/plataforma | Terraform: bootstrap da conta, rede, EKS, IAM, instalação inicial do Argo CD | — | 1 |
| `guetto-gitops` | Plataforma (apps via PR + CODEOWNERS) | Estado desejado do cluster: add-ons da plataforma e `apps/<app>/<env>` | chart `guetto-app` (versão OCI) | 2 |
| `guetto-charts` | Plataforma | Chart `guetto-app` com schema e testes; publicado como OCI no GHCR com semver | — | 3 |
| `guetto-ci` | Plataforma | Reusable workflows (build, test, scan, push, bump de tag) com tags de versão | — | 3 |
| `template-go-service` / `template-python-worker` | Plataforma | Template repositories do golden path | `guetto-ci` | 3 |
| `sample-*` | "Times de produto" | Aplicações de exemplo criadas a partir dos templates | `guetto-ci`, `guetto-gitops` (via PR/bump) | 3 |

Argo CD só tem acesso de leitura ao `guetto-gitops` (e ao registry OCI do chart).

## Contratos entre repositórios (o que precisa ser desenhado)

- **infra → gitops:** valores gerados pelo Terraform (nome do cluster, ARNs de roles, região, account ID) chegam ao
  Argo CD como labels/annotations do *cluster secret* (padrão **GitOps Bridge**), lidos por ApplicationSets.
  Nenhum ARN é copiado à mão para o `guetto-gitops` — isso preserva a portabilidade de conta (ADR-0003).
- **charts → gitops:** `guetto-gitops` referencia o chart por **versão semver**; atualização de versão é PR.
- **ci → apps:** apps chamam `guetto-ci/.github/workflows/x.yml@vN`; breaking change = nova major.
- **apps → gitops:** CI da app abre commit/PR de bump de tag via **GitHub App** com permissão mínima.

## Consequências

- Positivas: aprendizado de versionamento, contratos e menor privilégio entre repos; fronteiras realistas.
- Negativas: uma mudança de ponta a ponta pode exigir 2–3 PRs; mais repos para manter atualizados.
- Mitigação: repositórios criados **somente na fase em que são necessários**; nada de repos vazios.
- Revisar se o overhead de coordenação impedir progresso nas fases (sinal: muitas mudanças sempre acopladas entre dois repos
  → talvez a fronteira esteja errada e os repos devam ser unidos).

## Documentação

Documentação **transversal** fica em `.github/docs`. Cada repositório tem README próprio com: propósito, como usar,
como testar e link para os ADRs relevantes. Decisões locais a um repo podem ter `docs/adr/` no próprio repo.

## Referências

- [Argo CD Best Practices — separar repositório de config do de código-fonte](https://argo-cd.readthedocs.io/en/stable/user-guide/best_practices/) — fato documentado
- [GitOps Bridge](https://github.com/gitops-bridge-dev/gitops-bridge) e [módulo Terraform](https://github.com/gitops-bridge-dev/terraform-helm-gitops-bridge)
- [Akuity — integrando Terraform com Argo CD](https://akuity.io/blog/yet-another-take-on-integrating-terraform-with-argo-cd)
