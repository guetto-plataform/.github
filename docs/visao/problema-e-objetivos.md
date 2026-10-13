# Problema e objetivos

> Estado: **rascunho** — a ser refinado na Fase 0.

## Objetivo do projeto

Laboratório de **aprendizado de Platform Engineering**. O portfólio é consequência, não objetivo:
nenhuma peça entra na arquitetura só para "impressionar".

## Problema que a plataforma resolve

Um desenvolvedor de produto que quer colocar um serviço no ar precisa hoje dominar Kubernetes, AWS, IAM,
pipelines, observabilidade e segredos. A Guetto Platform oferece um **contrato**:

> "Descreva sua aplicação em um arquivo, faça push, e ela roda com deploy, rollback,
> telemetria e guardrails padronizados — sem escrever YAML de Kubernetes."

## Persona

**Dev de produto** — sabe programar (Go ou Python), sabe usar Git e GitHub, não quer (nem deveria precisar)
conhecer detalhes de EKS, IAM ou Prometheus para ter um serviço saudável em produção.

Como o projeto não tem usuários reais, a persona é exercitada por **aplicações de exemplo** com necessidades
diferentes (uma API em Go, um worker em Python).

## Métricas de sucesso do MVP (proposta — calibrar)

| Métrica | Alvo |
|---|---|
| Ambiente inteiro do zero (`make up`) | ≤ 30 min, sem passos manuais |
| Derrubar tudo (`make down`) sem recursos órfãos na conta | 100% das vezes |
| Serviço novo do zero até rodando em `dev` | ≤ 15 min |
| Configuração escrita pelo dev | 1 arquivo, ≤ 20 linhas |
| Rollback em `prod` | ≤ 5 min via `git revert` |
| Falha injetada diagnosticável via runbook | 3 de 3 cenários |
| Custo por sessão de estudo de 4h | a medir (estimativa em [custos](../arquitetura/custos.md)) |

## Fora do escopo (explicitamente)

- Reproduzir o Fury (Mercado Libre) ou qualquer plataforma de grande escala.
- Multi-cluster, portal de desenvolvedores, CLI/API própria e IA no MVP.
- Dados persistentes entre sessões (o ambiente é efêmero).
