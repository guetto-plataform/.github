# ADR-0011: Bootstrap da conta executado localmente, com state migrado e operação por scripts bash

- **Status:** Aceito
- **Data:** 2026-10-13
- **Fase do roadmap:** 1

## Contexto

O ADR-0003 (Proposto) definiu a camada `00-bootstrap` (bucket do state, Budgets, OIDC do GitHub) com state local.
Ao iniciar a Fase 1 surgiram três perguntas:

1. O bootstrap pode rodar pela esteira, de modo que trocar de conta seja "selecionar outra conta" num workflow?
2. O state do bootstrap fica local ou dentro do bucket que ele cria?
3. A interface de operação (`up`/`down`) é um Makefile (citado nos ADRs 0002 e 0003) ou scripts?

Restrições relevantes, verificadas em 2026-10-13:

- Numa conta nova não existe OIDC provider nem role: a esteira **não tem como se autenticar** sem chaves de longa
  duração ou sem algo criado manualmente antes (que já é o bootstrap).
- O caminho corporativo (conta de gerenciamento + AWS Organizations) torna a conta inelegível a novos créditos do
  Free Tier ([FAQ](https://aws.amazon.com/free/free-tier-faqs/)) — fato documentado.
- Os créditos são para clientes que nunca tiveram conta AWS ([FAQ](https://aws.amazon.com/free/free-tier-faqs/)).
  Logo, a troca de conta deve ser rara (0–2 vezes na vida do projeto) — avaliação.

## Alternativas consideradas

| Alternativa | Prós | Contras | Custo operacional |
|---|---|---|---|
| Bootstrap pela esteira com chaves de admin em secret do GitHub | "Um clique" por conta | Credencial permanente e de poder total fora da AWS — o que o OIDC elimina | Rotação de chaves |
| Bootstrap pela esteira via conta de gerenciamento (Organizations) | Padrão corporativo (account vending) | Afeta os créditos; complexidade muito acima do problema | Alto |
| **Bootstrap local por humano, esteira escolhe a conta via GitHub Environment** | Sem credencial permanente na esteira; troca de conta = 1 comando + escolher o environment | Passo manual por conta | Baixo |
| State do bootstrap local (`accounts/<conta>/`) | Primeiro apply e destroy triviais | Perde-se com o notebook (recuperável por `import`) | Baixo |
| **State do bootstrap migrado para o próprio bucket** | Nada importante fica no notebook; versionado | Primeiro apply exige override para backend local; destroy exige migrar de volta | Baixo (roteirizado) |
| Makefile como interface | Ponto de entrada único, dependências nativas | Ruim para lógica; `make` ausente no Windows | Baixo |
| **Scripts bash** | Linguagem completa; bash é competência pedida no mercado; roda no Git Bash e nos runners Linux | Sem grafo de dependências nem índice natural (resolver com convenção e `--help`) | Baixo |
| just / Task | Task runners modernos | Dependência menos conhecida para poucos comandos | Baixo |

## Decisão

1. **O `00-bootstrap` é aplicado somente por humano, localmente, uma vez por conta.** A esteira nunca aplica esta camada.
2. **State do bootstrap migrado:** primeiro apply com backend local (override temporário, não versionado), depois
   `terraform init -migrate-state` para `s3://<bucket da conta>/00-bootstrap/terraform.tfstate`.
3. **A esteira escolhe a conta por GitHub Environment** (`aws-<conta>`), com as variáveis `AWS_ROLE_ARN` e
   `TF_STATE_BUCKET` publicadas pelo próprio bootstrap. A trust policy da role usa o `sub` no formato de environment,
   com IDs imutáveis (repositórios criados após 15/07/2026).
4. **A role da CI não acessa o prefixo `00-bootstrap/`** do bucket — a esteira não pode alterar o state da camada
   que define as permissões dela.
5. **Proteção contra conta errada:** `allowed_account_ids` no provider AWS, com o ID vindo de `accounts/<conta>/`
   (não versionado).
6. **Interface de operação em scripts bash** em `scripts/`, sem Makefile. As menções a `make up` / `make down` nos
   ADRs 0002 e 0003 passam a significar os scripts equivalentes; a decisão de fundo (ambiente efêmero) não muda.
7. **Um único AWS Budget** de custo **bruto** (créditos não abatem o valor), **acumulado** (período `CUSTOM` cobrindo
   a validade dos créditos), limite **US$ 150**, com avisos em **US$ 75** e **US$ 150**. O alerta é tardio para
   "ambiente esquecido ligado" (~US$ 7/dia); essa proteção é responsabilidade do reaper.
8. **Bucket do state no namespace regional da conta** (`<prefixo>-<account-id>-<região>-an`): nome previsível que
   nenhuma outra conta pode criar, nem após exclusão
   ([S3 — namespaces](https://docs.aws.amazon.com/AmazonS3/latest/userguide/gpbucketnamespaces.html)).

## Consequências

- Positivas: nenhuma credencial permanente na esteira; o notebook é descartável; troca de conta documentada e curta.
- Negativas / dívidas assumidas:
  - O acesso local ainda usa IAM user com chaves de longa duração. IAM Identity Center exige *organization instance*
    ([account instance não dá acesso a contas](https://docs.aws.amazon.com/singlesignon/latest/userguide/account-instances-identity-center.html)),
    o que esbarra nos créditos. Alternativa no backlog: `aws login` (credenciais temporárias a partir do login de
    console, opção recomendada pela [doc do CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-authentication.html)).
  - Scripts bash no Windows exigem cuidado: finais de linha LF (`.gitattributes`) e executar no **Git Bash** —
    o `bash` do PATH do Windows é o do WSL.
- O que faria esta decisão ser revista: adoção de AWS Organizations; mais de uma pessoa operando o ambiente;
  necessidade de recriar contas com frequência.

## Como validar

- Bootstrap numa conta: `terraform state list` funciona com o backend S3 e nenhum `.tfstate` fica no disco.
- Rodar o bootstrap com o profile de outra conta falha no `plan` por `allowed_account_ids`.
- Workflow no environment `aws-<conta>` executa `aws sts get-caller-identity`; o mesmo workflow sem o environment falha.
- A role da CI recebe `AccessDenied` ao ler `00-bootstrap/terraform.tfstate`.

## Referências

- [AWS Free Tier FAQ](https://aws.amazon.com/free/free-tier-faqs/) — fato documentado
- [GitHub — OIDC na AWS](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws) — fato documentado
- [GitHub — subject claims imutáveis](https://github.blog/changelog/2026-04-23-immutable-subject-claims-for-github-actions-oidc-tokens) — fato documentado
- [Terraform — backend S3 (`use_lockfile`)](https://developer.hashicorp.com/terraform/language/backend/s3) — fato documentado
- [AWS Budgets API — TimeUnit, Metrics, FilterExpression](https://docs.aws.amazon.com/aws-cost-management/latest/APIReference/API_budgets_Budget.html) — fato documentado
- Padrão "bootstrap manual, depois a CI assume": [exemplo 1](https://dev.to/jajera/github-actions-to-aws-oidc-integration-setup-51me), [exemplo 2](https://dev.to/rmendoza/-one-click-deployments-1-bootstrap-a-gitops-ready-aws-terraform-backend-with-github-actions-49n2) — opinião/prática comunitária
