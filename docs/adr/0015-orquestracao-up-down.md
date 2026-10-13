# ADR-0015: Orquestração do `up`/`down` — contrato por SSM, limpeza em camadas e scripts únicos para local e CI

- **Status:** Aceito
- **Data:** 2026-10-13
- **Fase do roadmap:** 1

## Contexto

O ambiente sobe e desce por comando (ADR-0002) em camadas com states separados (ADR-0003), e o reaper executa o
`down` no GitHub Actions (ADR-0014). Três problemas precisam de decisão:

1. Como o `20-cluster` descobre a rede criada pelo `10-network`.
2. Como remover recursos AWS criados por controllers do Kubernetes (LBs, security groups, volumes EBS, ENIs), que o
   Terraform não conhece e que fazem o destroy da VPC falhar (`DependencyViolation`) ou continuam cobrando.
3. Como o mesmo `down` roda localmente e na CI, onde `accounts/<conta>/` não existe (não é versionado).

Fatos: a HashiCorp desaconselha `terraform_remote_state` porque o leitor precisa de acesso ao state inteiro, e
recomenda publicar dados em local separado, como o SSM Parameter Store
([doc](https://developer.hashicorp.com/terraform/language/state/remote-state-data)). A biblioteca de módulos já
publica valores em SSM (`iam-irsa`, `eks-karpenter`, `ecr`, `eks-aws-lb-controller`).

## Alternativas consideradas

| Alternativa | Prós | Contras | Custo operacional |
|---|---|---|---|
| `terraform_remote_state` | Simples, nativo | Acesso ao state inteiro; acopla ao formato do backend | Baixo |
| Data sources por tag | Nenhum recurso extra | Contrato implícito; quebra silenciosamente se tags mudarem | Baixo |
| **Parâmetros SSM** | Contrato explícito; recomendado pela HashiCorp; mesmo padrão da biblioteca e base para o contrato infra → gitops | Alguns recursos a mais (standard: gratuitos) | Baixo |
| Limpeza só via Kubernetes | Cada controller limpa o que criou | Controller travado = órfãos | Baixo |
| Limpeza só por varredura na AWS | Funciona com cluster quebrado | Força bruta; risco com tags mal escolhidas | Médio |
| **Limpeza em camadas** | Caminho correto + verificação auditável; cobre o "script pós-down" de custos.md | Mais código | Médio |
| Scripts separados para local e CI | Cada um simples | Dois caminhos; o do reaper não é o testado à mão | Alto |
| **Config por variáveis de ambiente, script único** | O `down` do reaper é o mesmo testado localmente | Carregador local para `accounts/<conta>/` | Baixo |

## Decisão

1. **Contrato entre camadas por SSM Parameter Store:** o `10-network` publica, por exemplo,
   `/guetto/network/vpc-id` e `/guetto/network/private-subnet-ids`; o `20-cluster` lê com `data "aws_ssm_parameter"`.
   Os parâmetros nascem e morrem com a camada que os publica.
2. **Limpeza em camadas no `down`:**
   1. Com o cluster vivo: remover recursos Kubernetes que criam recursos AWS (Ingress, Services `LoadBalancer`, PVCs;
      a partir da Fase 2, Applications do Argo CD em cascata) e aguardar os controllers limparem.
   2. `terraform destroy` do `20-cluster`.
   3. **Varredura por tag**: remove apenas órfãos comprovadamente criados pelo cluster (tags dos controllers/CNI).
   4. `terraform destroy` do `10-network`.
   5. **Verificação final**: lista recursos com tags do projeto que ainda existam e **falha** se houver algum.
3. **Scripts únicos** `scripts/up.sh` e `scripts/down.sh`, configurados por **variáveis de ambiente**
   (`AWS_REGION`, `TF_STATE_BUCKET`…). Localmente, um carregador lê `accounts/<conta>/` e as preenche; na CI vêm do
   GitHub Environment. Funções comuns em `scripts/lib/common.sh` (o `bootstrap.sh` passa a usá-las).
4. O `down` é **idempotente**: com nada de pé, termina sem alterações (exigido pela execução de segurança do reaper).

## Consequências

- Positivas: um único caminho de código para humano e reaper; `down` robusto a controllers travados; contrato entre
  camadas visível no código.
- Negativas / dívidas assumidas: a varredura depende de convenção de tags dos controllers — precisa ser revista a
  cada controller novo (Fase 2: LB Controller, EBS CSI).
- O que faria esta decisão ser revista: órfãos recorrentes não cobertos pela varredura; adoção de outra ferramenta
  de orquestração.

## Como validar

- `up` seguido de `down` três vezes sem órfãos (critério da Fase 1 no roadmap).
- `down` com nada de pé: sucesso, sem alterações.
- `down` após apagar um LB "à mão" do cluster: a varredura ou a verificação final detecta/remediam.

## Referências

- [Terraform — `terraform_remote_state` e alternativas](https://developer.hashicorp.com/terraform/language/state/remote-state-data) — fato documentado
- [ENI presa após destruir EKS (AWS re:Post)](https://repost.aws/questions/QURdzqTaExQoCLRBiOAyB7YQ/cannot-delete-eni-attached-to-destroyed-eks) — relato da comunidade
