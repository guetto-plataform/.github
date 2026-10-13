# ADR-0010: Reutilizar a biblioteca `terraform-aws-modules` consumida por versão

- **Status:** Aceito
- **Data:** 2026-10-13
- **Fase do roadmap:** 1

## Contexto

O autor mantém a biblioteca [vitorfprado/terraform-aws-modules](https://github.com/vitorfprado/terraform-aws-modules)
(tag `v1.0.0`) com módulos de VPC, EKS, IRSA, AWS Load Balancer Controller (só IAM), Karpenter, ECR, RDS, SQS, entre outros.

Avaliação dos módulos relevantes para a Fase 1:

| Aspecto | Situação |
|---|---|
| Estrutura | Boa: um módulo por pasta, `example/`, README e `doc/explain.md` |
| VPC | Subnets públicas/privadas, NAT único ou por AZ, tags de subnet para ELB por variável |
| EKS | Access entries (API), KMS para secrets, log group gerenciado pelo Terraform, add-ons, EBS CSI |
| LB Controller | Cria **apenas a IAM role** — a instalação fica para o GitOps, coerente com o ADR-0003 |
| Contrato entre stacks | Publica ARNs no SSM Parameter Store — alternativa ao GitOps Bridge a avaliar |

## Alternativas consideradas

| Alternativa | Prós | Contras |
|---|---|---|
| **Consumir a biblioteca por tag (`?ref=vX.Y.Z`)** | Realista (biblioteca interna versionada); reaproveita trabalho; força semver e changelog | Mudanças exigem PR + release na biblioteca |
| Copiar os módulos para o `guetto-infra` | Rápido de alterar | Duplicação; perde o exercício de versionamento |
| Módulos da comunidade (`terraform-aws-modules/*` do registry) | Muito maduros | Abstração enorme; esconde os mecanismos que se quer aprender |
| Escrever do zero | Aprendizado máximo | Retrabalho do que já existe |

## Decisão

O `guetto-infra` consome a biblioteca por **tag fixa**, ex.:
`source = "git::https://github.com/vitorfprado/terraform-aws-modules.git//eks?ref=v1.0.0"`.
A biblioteca permanece no perfil pessoal (é reutilizável fora da Guetto Platform).
Ajustes necessários entram como **PR + nova versão** na biblioteca, nunca como cópia local.

## Ajustes identificados para o uso na Guetto (backlog da biblioteca)

1. **Versão do Kubernetes:** o default `1.32` está em *extended support* desde 23/03/2026 (custo ~6× no control plane).
   O `guetto-infra` deve fixar uma versão em standard support (1.35 ou 1.36 em out/2026) e a biblioteca deve
   remover ou atualizar o default.
2. **Ambiente efêmero:** cada `up` cria uma nova KMS key; após `down` ela fica *pending deletion* (sem cobrança
   durante a espera, conforme a página de preços). Usar `kms_key_deletion_window_in_days = 7` (mínimo).
3. **Logs do control plane:** retenção padrão de 90 dias e log `audit` habilitado geram custo de ingestão;
   reduzir retenção e habilitar `audit` só quando for estudado.
4. **EKS Pod Identity:** os módulos usam IRSA. A AWS recomenda Pod Identity para novos workloads, e ele não
   depende do OIDC issuer (que muda a cada recriação do cluster). Candidato a novo módulo (`iam-pod-identity`).
5. **VPC Gateway Endpoint para S3:** não existe no módulo `vpc`.
6. **Pinagem de providers:** módulos usam `>= 5.40` (correto para biblioteca); o `guetto-infra` fixa versões
   e **commita** o `.terraform.lock.hcl` (a biblioteca o ignora, o que é adequado só para bibliotecas).

## Consequências

- Experiência realista de "time de plataforma consome biblioteca interna".
- Cada ajuste vira PR na biblioteca + release — atrito intencional e documentado.

## Referências

- [Calendário de versões do EKS](https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html)
- [Preço do extended support do EKS](https://aws.amazon.com/pt/blogs/containers/amazon-eks-extended-support-for-kubernetes-versions-pricing/)
- [EKS Pod Identity](https://docs.aws.amazon.com/en_us/eks/latest/userguide/pod-identities.html) · [Best practices — cluster access](https://docs.aws.amazon.com/eks/latest/best-practices/cluster-access-management.md)
- [Preços do KMS](https://aws.amazon.com/kms/pricing/)
