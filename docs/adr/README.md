# Architecture Decision Records

Cada ADR registra **uma** decisão: contexto, alternativas, decisão e consequências.
Modelo em [0000-template.md](0000-template.md).

## Ciclo de vida

`Proposto` → `Aceito` → (opcional) `Substituído por ADR-XXXX` ou `Descontinuado`.

ADR aceito não é editado (exceto correções de texto). Mudou de ideia? Novo ADR que substitui o antigo —
o histórico de decisões é parte do aprendizado.

## Índice

| # | Título | Status |
|---|---|---|
| [0001](0001-plataforma-como-produto-e-escopo-do-mvp.md) | Plataforma como produto e escopo enxuto do MVP | Aceito |
| [0002](0002-aws-com-ambiente-efemero.md) | AWS com ambiente efêmero (up/down via IaC) | Aceito |
| [0003](0003-terraform-em-camadas-e-portabilidade-de-conta.md) | Terraform em camadas e portabilidade de conta | Proposto |
| [0004](0004-cluster-unico-com-ambientes-por-namespace.md) | Cluster único com ambientes por namespace | Proposto |
| [0005](0005-config-de-deploy-no-repo-da-plataforma.md) | Configuração de deploy no repositório da plataforma | Aceito |
| [0006](0006-external-secrets-operator.md) | External Secrets Operator para segredos | Aceito (backend: Proposto) |
| [0007](0007-go-e-python.md) | Go e Python como linguagens do projeto | Aceito |
| [0008](0008-ghcr-como-registry.md) | GHCR como registry de imagens | Proposto |
| [0009](0009-multirepo-por-fronteira-de-responsabilidade.md) | Multirepo separado por fronteira de responsabilidade | Aceito |
| [0010](0010-reutilizar-biblioteca-de-modulos-terraform.md) | Reutilizar a biblioteca `terraform-aws-modules` por versão | Aceito |
| [0011](0011-bootstrap-local-state-migrado-e-scripts-bash.md) | Bootstrap local com state migrado; operação por scripts bash | Aceito |
| [0012](0012-acesso-da-ci-a-aws-via-oidc.md) | Acesso da CI à AWS via OIDC, role única por conta e guardrails por Deny | Aceito |
| [0013](0013-desenho-da-vpc.md) | Desenho da VPC: 3 AZs, `10.20.0.0/16`, seleção de AZ por ID, S3 Gateway Endpoint | Aceito |
| [0014](0014-reaper-horario-fixo.md) | Reaper diário via GitHub Actions às 23h (Brasília), com execução de segurança | Aceito |
| [0015](0015-orquestracao-up-down.md) | Orquestração do up/down: contrato por SSM, limpeza em camadas, scripts únicos | Aceito |
