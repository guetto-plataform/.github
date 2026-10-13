# Arquitetura

> Estado: **esboço**. Diagramas C4 formais serão criados na Fase 0 (contexto) e na Fase 3 (containers).

## Visão geral (proposta)

```
 GitHub (org)                                         AWS (conta intercambiável)
 ┌──────────────────────────────┐                    ┌──────────────────────────────────────────┐
 │ .github       (docs, perfil) │                    │ Camada 0 - bootstrap (persistente, ~US$0) │
 │ guetto-infra  (Terraform) ───┼── terraform apply ►│   S3 (state) · Budgets · OIDC do GitHub   │
 │                              │  + cluster secret  │                                          │
 │                              │   (GitOps Bridge)  │ Camadas 1-2 - efêmeras (só durante testes)│
 │ guetto-gitops (estado) ◄─────┼──── pull ──────────┼── Argo CD                                │
 │ guetto-charts (OCI semver) ──┼─► GHCR ◄── pull ───┼── Argo CD                                │
 │ guetto-ci (reusable wf @vN)  │                    │   VPC · EKS · IAM (Pod Identity)          │
 │ template-* / sample-* ──CI──►│ GHCR (imagens)     │   External Secrets ◄── SSM/Secrets Mgr    │
 │        └── bump de tag ─────►│ guetto-gitops      │   LB Controller · observabilidade         │
 └──────────────────────────────┘                    │   namespaces dev / prod (apps)            │
                                                     └──────────────────────────────────────────┘
```

## Princípios

1. **Efêmero por padrão** — só existe custo durante sessões de estudo. `make up` / `make down` reconstroem tudo.
2. **Conta AWS intercambiável** — nenhum ID de conta, ARN ou nome de bucket fixo no código; trocar de conta
   = novo profile + novo bootstrap + novo arquivo de backend. Ver [ADR-0003](../adr/0003-terraform-em-camadas-e-portabilidade-de-conta.md).
3. **Terraform até o "cluster pronto + Argo CD instalado"**; tudo dentro do cluster é GitOps.
4. **Artefatos fora da AWS** (imagens no GHCR, código no GitHub) para que a troca de conta não exija migração.
5. **Plataforma como produto** — o que se entrega ao dev é o contrato (`guetto-app`), não as ferramentas.

## Documentos previstos

- `contexto.md` — C4 nível 1 (Fase 0)
- `containers.md` — C4 nível 2 (Fase 3)
- `implantacao.md` — topologia AWS/EKS, redes, IAM (Fase 1)
- [custos.md](custos.md) — estimativas e medições reais
