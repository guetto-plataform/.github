# ADR-0006: External Secrets Operator para segredos

- **Status:** Aceito (escolha do backend: Proposto)
- **Data:** 2026-10-13
- **Fase do roadmap:** 2

## Contexto

GitOps não deve conter valores de segredo. As aplicações precisam de segredos como `Secret` do Kubernetes.

## Decisão

External Secrets Operator (ESO), autenticado na AWS via **EKS Pod Identity/IRSA** (sem chaves estáticas).
`ExternalSecret` fica no Git; o valor fica na AWS.

## Backend — a decidir

| Backend | Custo | Observação |
|---|---|---|
| SSM Parameter Store (standard, SecureString) | Gratuito no tier standard | Suficiente para o lab |
| Secrets Manager | Por segredo/mês | Rotação nativa; mais "corporativo" |

Recomendação inicial: **SSM Parameter Store** (custo zero, mesmo modelo de uso pelo ESO).

## Consequência para a portabilidade de conta

Segredos vivem na conta AWS e somem na troca de conta. É preciso uma **fonte de semeadura** reprodutível
(ex.: script que lê um arquivo local fora do Git, ou SOPS). Decidir na Fase 2.
