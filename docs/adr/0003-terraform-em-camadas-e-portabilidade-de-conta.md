# ADR-0003: Terraform em camadas, com conta AWS intercambiável

- **Status:** Proposto
- **Data:** 2026-10-13
- **Fase do roadmap:** 1

## Contexto

Requisitos: (1) subir e derrubar tudo por IaC; (2) trocar de conta AWS quando os créditos acabarem sem reescrever código;
(3) evitar o anti-padrão de gerenciar cluster e recursos Kubernetes na mesma configuração Terraform.

## Proposta

### Camadas (um state por camada)

| Camada | Conteúdo | Ciclo de vida |
|---|---|---|
| `00-bootstrap` | Bucket S3 do state, AWS Budgets, OIDC provider do GitHub + role da CI | **Persistente**, uma vez por conta, custo ~zero. Execução e state: ver ADR-0011 |
| `10-network` | VPC, subnets, security groups | Efêmera |
| `20-cluster` | EKS, node group, add-ons, IAM (Pod Identity) para controllers, bootstrap do Argo CD | Efêmera |

Tudo dentro do cluster depois do Argo CD é **GitOps**, não Terraform.

### Portabilidade de conta

- Nenhum account ID, ARN ou nome de bucket fixo: usar `data.aws_caller_identity` e variáveis.
- Seleção de conta por **AWS profile** + arquivos por conta:
  `accounts/<nome>/backend.hcl` e `accounts/<nome>/account.tfvars`.
- Trocar de conta = criar profile → aplicar `00-bootstrap` → `terraform init -backend-config=accounts/<nova>/backend.hcl`.
- Artefatos fora da AWS (imagens no GHCR — ADR-0008) eliminam migração de dados.
- Segredos precisam ser **re-semeados** na conta nova (ver ADR-0006).

### Rede: subnets privadas + NAT Gateway gerenciado (decidido em 2026-10-13)

Prioridade do autor: ambiente o mais realista possível. Portanto:

- Nós e pods em **subnets privadas**; subnets públicas apenas para load balancers e NAT.
- **NAT Gateway gerenciado** (não instância NAT nem fck-nat).
- **Um único NAT por padrão** (`single_nat_gateway = true`), como é comum em ambientes não produtivos;
  NAT por AZ fica disponível por variável para experimentos de alta disponibilidade.
  Trade-off aceito: falha da AZ do NAT corta a saída à internet das outras AZs.
- **VPC Gateway Endpoint para S3** (gratuito) para reduzir tráfego processado pelo NAT — ainda não existe na
  biblioteca de módulos (ver ADR-0010).
- Endpoint da API do EKS: público restrito ao IP do operador + privado (decidir na Fase 1).

### Lock do state

S3 com lock nativo (`use_lockfile`), sem DynamoDB — **verificar versão mínima do Terraform** antes de adotar.

## Alternativas a registrar

- Terragrunt (DRY entre contas) — provavelmente excessivo para uma conta por vez.
- Workspaces do Terraform por conta — mistura states de contas diferentes no mesmo backend; rejeitar.
- OpenTofu — viável; Terraform escolhido pelo autor.

## Como validar

- `make up` numa conta nova após apenas criar o profile e rodar o bootstrap.
- `make down` seguido de varredura sem recursos órfãos.

## Referências

- [Não usar o provider Kubernetes no mesmo módulo que cria o cluster (ITNext)](https://itnext.io/terraform-dont-use-kubernetes-provider-with-your-cluster-resource-d8ec5319d14a)
- [Pulumi citando a documentação do provider Kubernetes](https://www.pulumi.com/blog/terraform-kubernetes/)
