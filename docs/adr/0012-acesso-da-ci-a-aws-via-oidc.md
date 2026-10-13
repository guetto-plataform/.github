# ADR-0012: Acesso da CI à AWS via OIDC, com role única por conta e guardrails por Deny

- **Status:** Aceito
- **Data:** 2026-10-13
- **Fase do roadmap:** 1

## Contexto

Os workflows do `guetto-infra` (`up`/`down`, reaper) precisam operar na AWS. O ADR-0011 definiu que a esteira
escolhe a conta por GitHub Environment (`aws-<conta>`) e que o bootstrap cria o acesso. Falta decidir como a CI se
autentica, quantas roles existem e com quais permissões.

Fatos verificados em 2026-10-13:

- GitHub Actions emite um JWT por job (`id-token: write`); a AWS troca esse token por credenciais temporárias via
  `sts:AssumeRoleWithWebIdentity`. A condição sobre `sub` é o que impede outros repositórios de assumir a role
  ([GitHub](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws)).
- Repositórios criados após 15/07/2026 recebem `sub` com IDs imutáveis:
  `repo:<org>@<org-id>/<repo>@<repo-id>:...`
  ([changelog](https://github.blog/changelog/2026-04-23-immutable-subject-claims-for-github-actions-oidc-tokens)).
  Job com `environment:` recebe `...:environment:<nome>` no lugar de `...:ref:<ref>`.
- `aws-actions/configure-aws-credentials` **não** mascara o account ID por padrão (`mask-aws-account-id`), e logs de
  repositório público são públicos.
- Criar EKS + rede exige permissões amplas (IAM, EC2, EKS, KMS, Logs, Auto Scaling).

## Alternativas consideradas

| Alternativa | Prós | Contras | Custo operacional |
|---|---|---|---|
| Chaves de IAM user em secrets | Simples | Credencial permanente fora da AWS; rotação manual | Alto (risco) |
| **OIDC + 1 role por conta (apply)** | Sem segredo permanente; uma role para os workflows que existem | Plan em PR não tem role só-leitura (ainda não há esse workflow) | Baixo |
| OIDC + roles de plan e apply | Menor privilégio para PRs | Role sem consumidor hoje | Baixo |
| Permissões least-privilege customizadas | Menor privilégio real | Dezenas de ações descobertas por tentativa e erro; quebra a cada recurso novo | Alto |
| **AdministratorAccess + Denies explícitos** | Honesto quanto à necessidade; protege a camada persistente de acidentes | Denies não impedem escalação deliberada (ver Consequências) | Baixo |
| Permission boundary obrigatório em roles criadas | Barreira real contra escalação | Exige suporte a `permissions_boundary` na biblioteca de módulos (ADR-0010) | Médio |
| Environment e variáveis criados por `gh api` em script | Sem provider extra | Configuração de segurança (branch policy) fora do código e da revisão | Baixo |
| **Environment criado pelo provider `integrations/github` no bootstrap** | Declarativo; branch policy versionada e revisada | Segundo provider; token local (`GITHUB_TOKEN`) | Baixo |

## Decisão

1. **OIDC provider** do GitHub por conta, `aud = sts.amazonaws.com`, sem thumbprint (validação pela cadeia de CAs da AWS).
2. **Uma role por conta** (`guetto-github-actions`), com trust policy exigindo `aud` e
   `sub = repo:guetto-plataform@<org-id>/guetto-infra@<repo-id>:environment:aws-<conta>` (`StringEquals`, sem curingas).
   Sessão máxima de 2h. Role de plan só-leitura entra junto com o workflow de plan em PR.
3. **Permissões:** `AdministratorAccess` + política inline de **Deny** para: a própria role e o OIDC provider (`iam:*`),
   o budget (`budgets:ModifyBudget`), o prefixo `00-bootstrap/` do bucket de state e a configuração do bucket.
4. **GitHub Environment `aws-<conta>`** gerenciado pelo bootstrap (provider `integrations/github`), aceitando
   **somente o branch `main`**. `AWS_ROLE_ARN` e `TF_STATE_BUCKET` são **secrets** (contêm o account ID e o GitHub os
   mascara); `AWS_REGION` é variável. Isso refina o ADR-0011, que falava em variáveis.
5. Workflows usam `mask-aws-account-id: true` e fixam actions de terceiros por **SHA de commit**.

## Consequências

- Positivas: nenhuma credencial permanente na esteira; acesso limitado a um repositório, um branch e um environment;
  troca de conta = novo environment.
- Negativas / dívidas assumidas:
  - **Risco aceito:** a role é admin. Os Denies são quebra-molas, não muros — uma role admin pode criar outra role
    sem as restrições e assumi-la. O controle efetivo hoje é a trust policy + a branch policy do environment.
    Mitigação planejada: permission boundary obrigatório (Fase 5 — guardrails).
  - Quem tem escrita no `main` tem, na prática, admin na conta AWS. Hoje só o autor; revisar se houver colaboradores
    (proteção de branch + revisão obrigatória).
  - O ID numérico da org é variável no código (o data source `github_organization` exige escopo `read:user`); é
    imutável por definição.
- O que faria esta decisão ser revista: colaboradores com escrita no repo; workflow de plan em PR; adoção de
  permission boundary.

## Como validar

- `aws-oidc-smoke-test` a partir do `main`: job com environment assume a role e `get-caller-identity` mostra
  `assumed-role/guetto-github-actions/...`.
- O mesmo workflow a partir de outro branch: o GitHub bloqueia o job antes de emitir o token.
- O job sem environment mostra `sub` terminando em `:ref:refs/heads/main`, que não casa com a trust policy.

## Referências

- [GitHub — OIDC na AWS](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws) — fato documentado
- [GitHub — subject claims imutáveis](https://github.blog/changelog/2026-04-23-immutable-subject-claims-for-github-actions-oidc-tokens) — fato documentado
- [configure-aws-credentials — action.yml](https://github.com/aws-actions/configure-aws-credentials/blob/main/action.yml) — fato documentado
- [IAM — avaliação de políticas (Deny explícito prevalece)](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html) — fato documentado
- [IAM — permissions boundaries](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html) — fato documentado
