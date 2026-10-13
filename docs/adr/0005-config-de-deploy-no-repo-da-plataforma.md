# ADR-0005: Configuração de deploy no repositório da plataforma

- **Status:** Aceito
- **Data:** 2026-10-13
- **Fase do roadmap:** 2–3

> Nota (2026-10-13): com o [ADR-0009](0009-multirepo-por-fronteira-de-responsabilidade.md), o repositório da plataforma
> que guarda a configuração de deploy passa a ser o **`guetto-gitops`**. A decisão (config centralizada em repo da
> plataforma, ambientes por diretório) permanece.

## Contexto

O estado desejado de cada aplicação (values do chart `guetto-app` por ambiente, tag da imagem) precisa morar em algum lugar
que o Argo CD leia.

## Alternativas consideradas

| Alternativa | Prós | Contras |
|---|---|---|
| **`guetto-platform/gitops/apps/<app>/<env>.yaml`** | Um único repo para o Argo CD; ApplicationSet com git generator simples; governança por CODEOWNERS; onboarding = PR | CI das apps precisa de escrita cross-repo (GitHub App) |
| Values no repo da app (multi-source do Argo CD) | Dev controla tudo no próprio repo | Argo CD precisa acessar N repos; onboarding menos explícito; governança dispersa |
| Repo GitOps separado só para estado | Separação de permissões | Mais um repo para uma pessoa manter |

## Decisão

Primeira alternativa. Ambientes separados por **diretório/arquivo**, nunca por branch.
Bump de tag em `dev` por commit da CI (via GitHub App, não PAT pessoal); promoção para `prod` por PR.

## Quando revisar

Se o atrito de onboarding por PR for medido como alto, ou ao avaliar Kargo para promoção.

## Referências

- [Stop using branches for GitOps environments (Kapelonis)](https://octopus.com/blog/stop-using-branches-deploying-different-gitops-environments)
- [Kargo](https://akuity.io/what-is-kargo)
