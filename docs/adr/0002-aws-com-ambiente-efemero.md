# ADR-0002: Usar AWS desde o MVP, com ambiente efêmero

- **Status:** Aceito
- **Data:** 2026-10-13
- **Fase do roadmap:** 0

## Contexto

O autor dispõe de US$ 200 em créditos AWS e prefere não rodar o laboratório localmente.
O foco é aprendizado de Platform Engineering, e um ambiente em cloud expõe problemas reais (IAM, redes,
load balancers, custo) que um cluster local esconde.

## Alternativas consideradas

| Alternativa | Prós | Contras |
|---|---|---|
| Cluster local (kind/k3d) | Custo zero, rápido | Não exercita IAM, redes e Terraform de cloud; consome a máquina |
| AWS sempre ligada | Simples de operar | ~US$ 200/mês: consome todo o crédito em ~1 mês |
| **AWS efêmera (up/down por IaC)** | Custo só durante estudo; força automação completa e reprodutível | Tempo de subida (~15–20 min de EKS); estado não persiste entre sessões |
| k3s em EC2 | Sem taxa de control plane | Não é o que o mercado usa; mais operação manual |

## Decisão

AWS com EKS, **efêmera**: todo o ambiente sobe e desce por `make up` / `make down`. Nada que custe por hora
fica ligado fora de sessões de estudo. Ver estimativas e proteções em [custos.md](../arquitetura/custos.md).

## Consequências

- "Recriar do zero" vira teste contínuo da plataforma (prova real de GitOps e IaC).
- Dados (métricas, logs) se perdem entre sessões — aceitável no laboratório; reavaliar na fase de observabilidade.
- Obrigatório: Budgets, reaper agendado e verificação de órfãos após `down`.
- `terraform destroy` falha se o cluster criou recursos fora do Terraform (LBs, ENIs, volumes);
  o `make down` precisa remover os workloads/controllers **antes** do destroy.

## Contexto da conta (2026-10-13)

- Paid plan, região `us-east-1`. Excedente aos créditos é cobrado — Budgets e reaper são obrigatórios desde a Fase 1.
- Acesso local via AWS CLI profile (nome configurável; nunca fixo no código).
- O acesso atual usa um IAM user com chaves de longa duração. Avaliar migração para IAM Identity Center (credenciais
  temporárias) como item de segurança da Fase 1/5.

## Referências

- [AWS — novo Free Tier com US$ 200 em créditos](https://aws.amazon.com/about-aws/whats-new/2025/07/aws-free-tier-credits-month-free-plan/)
- [Recursos órfãos no destroy do EKS (issue HashiCorp)](https://github.com/hashicorp/learn-terraform-provision-eks-cluster/issues/60)
- [ENI presa após destruir EKS (AWS re:Post)](https://repost.aws/questions/QURdzqTaExQoCLRBiOAyB7YQ/cannot-delete-eni-attached-to-destroyed-eks)
